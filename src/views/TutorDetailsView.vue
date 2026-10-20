<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink, useRoute } from 'vue-router';

const API_URL = 'http://localhost:3000';
const route = useRoute();
const tutorId = route.params.id;

const tutor = ref(null);
const pets = ref([]);
const novoPet = ref({
  nome: '',
  especie: '',
});
const erro = ref('');
const carregando = ref(true);
const salvandoPet = ref(false);

function formatarCpfCnpj(valor) {
  if (!valor) {
    return '';
  }

  if (valor.includes('*')) {
    return valor;
  }

  const digitos = valor.replace(/\D/g, '');
  if (digitos.length === 11) {
    return `***.***.***-${digitos.slice(-2)}`;
  }
  if (digitos.length === 14) {
    return `**.***.***/****-${digitos.slice(-2)}`;
  }

  return valor;
}

function formatarTelefone(valor) {
  const digitos = (valor || '').replace(/\D/g, '');
  if (digitos.length === 10) {
    return digitos.replace(/(\d{2})(\d{4})(\d{4})/, '($1) $2-$3');
  }
  if (digitos.length === 11) {
    return digitos.replace(/(\d{2})(\d{5})(\d{4})/, '($1) $2-$3');
  }
  return valor || '-';
}

function formatarData(valor) {
  if (!valor) {
    return '';
  }

  const [ano, mes, dia] = valor.split('-');
  return dia && mes && ano ? `${dia}/${mes}/${ano}` : valor;
}

async function carregarDados() {
  carregando.value = true;

  try {
    const [respostaTutor, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores/${tutorId}`),
      fetch(`${API_URL}/pets?tutorId=${encodeURIComponent(tutorId)}`),
    ]);

    if (!respostaTutor.ok || !respostaPets.ok) {
      throw new Error('Não foi possível carregar os dados do tutor.');
    }

    tutor.value = await respostaTutor.json();
    pets.value = await respostaPets.json();
    erro.value = '';
  } catch (error) {
    console.error('Erro ao carregar dados do tutor:', error);
    erro.value = 'Não foi possível carregar os dados do tutor. Tente novamente.';
  } finally {
    carregando.value = false;
  }
}

async function salvarPet() {
  salvandoPet.value = true;

  try {
    const resposta = await fetch(`${API_URL}/pets`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nome: novoPet.value.nome.trim(),
        especie: novoPet.value.especie,
        tutorId: String(tutorId),
      }),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível cadastrar o pet.');
    }

    novoPet.value = { nome: '', especie: '' };
    await carregarDados();
  } catch (error) {
    console.error('Erro ao cadastrar pet:', error);
    erro.value = 'Não foi possível cadastrar o pet. Tente novamente.';
  } finally {
    salvandoPet.value = false;
  }
}

onMounted(carregarDados);
</script>

<template>
  <div>
    <header class="mb-4 d-flex flex-wrap justify-content-between align-items-center gap-3">
      <div>
        <h1 class="text-2xl font-bold">Detalhes do Tutor</h1>
        <p class="text-body-secondary mb-0">
          Visualize os dados do tutor e cadastre pets vinculados.
        </p>
      </div>
      <RouterLink class="btn btn-outline-primary" to="/tutores">
        <i class="bi bi-arrow-left me-1" aria-hidden="true"></i>
        Voltar para Tutores
      </RouterLink>
    </header>

    <div v-if="erro" class="alert alert-danger" role="alert">
      {{ erro }}
    </div>

    <section
      v-if="tutor"
      class="card shadow-sm mb-4"
      aria-labelledby="informacoes-tutor"
    >
      <div class="card-body p-4">
        <h2 id="informacoes-tutor" class="h4 mb-4">Informações do Tutor</h2>

        <div class="border rounded bg-body-tertiary p-3 mb-3">
          <h3 class="h6 text-body-secondary mb-3">Dados do Tutor</h3>
          <div class="row g-3">
            <div class="col-sm-6 col-lg-4">
              <div class="small fw-semibold text-body-secondary">ID</div>
              <div>{{ tutor.id }}</div>
            </div>
            <div class="col-sm-6 col-lg-4">
              <div class="small fw-semibold text-body-secondary">Nome Completo</div>
              <div>{{ tutor.nomeCompleto || tutor.nome }}</div>
            </div>
            <div v-if="tutor.cpfCnpj || tutor.cpf" class="col-sm-6 col-lg-4">
              <div class="small fw-semibold text-body-secondary">CPF / CNPJ</div>
              <div>{{ formatarCpfCnpj(tutor.cpfCnpj || tutor.cpf) }}</div>
            </div>
            <div class="col-sm-6 col-lg-4">
              <div class="small fw-semibold text-body-secondary">
                Telefone / Celular / WhatsApp
              </div>
              <div>{{ formatarTelefone(tutor.telefone) }}</div>
            </div>
            <div v-if="tutor.email" class="col-sm-6 col-lg-4">
              <div class="small fw-semibold text-body-secondary">E-mail</div>
              <div>{{ tutor.email }}</div>
            </div>
            <div
              v-if="tutor.dataNascimento || tutor.dataDeNascimento"
              class="col-sm-6 col-lg-4"
            >
              <div class="small fw-semibold text-body-secondary">Data de Nascimento</div>
              <div>
                {{ formatarData(tutor.dataNascimento || tutor.dataDeNascimento) }}
              </div>
            </div>
          </div>
        </div>

        <div
          v-if="tutor.endereco && Object.values(tutor.endereco).some(Boolean)"
          class="border rounded bg-body-tertiary p-3"
        >
          <h3 class="h6 text-body-secondary mb-3">Endereço</h3>
          <div class="row g-3">
            <div
              v-for="(rotulo, campo) in {
                rua: 'Rua',
                numero: 'Número',
                bairro: 'Bairro',
                cidade: 'Cidade',
                estado: 'Estado',
                cep: 'CEP',
              }"
              :key="campo"
              v-show="tutor.endereco[campo]"
              class="col-sm-6 col-lg-4"
            >
              <div class="small fw-semibold text-body-secondary">{{ rotulo }}</div>
              <div>{{ tutor.endereco[campo] }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section aria-labelledby="pets-tutor">
      <h2 id="pets-tutor" class="h4 mb-3">Pets do Tutor</h2>
      <div class="row g-4 align-items-start">
        <div class="col-12 col-lg-8">
          <div class="card shadow-sm">
            <div
              class="card-header bg-body d-flex justify-content-between align-items-center py-3"
            >
              <h3 class="h6 mb-0">Pets vinculados</h3>
              <span class="badge text-bg-primary">
                {{ pets.length }} {{ pets.length === 1 ? 'pet' : 'pets' }}
              </span>
            </div>

            <div v-if="carregando" class="card-body text-body-secondary">
              Carregando pets...
            </div>
            <div
              v-else-if="pets.length === 0 && !erro"
              class="card-body text-body-secondary"
            >
              Nenhum pet vinculado a este tutor.
            </div>
            <div v-else-if="pets.length > 0" class="table-responsive">
              <table class="table table-striped table-hover align-middle mb-0">
                <thead>
                  <tr>
                    <th scope="col">ID</th>
                    <th scope="col">Nome</th>
                    <th scope="col">Espécie</th>
                    <th scope="col" class="text-end">Ações</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="pet in pets" :key="pet.id">
                    <td>{{ pet.id }}</td>
                    <td>{{ pet.nome }}</td>
                    <td>{{ pet.especie }}</td>
                    <td class="text-end">
                      <RouterLink
                        class="btn btn-sm btn-outline-primary"
                        :to="`/pets/${pet.id}`"
                      >
                        Visualizar
                      </RouterLink>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div class="col-12 col-lg-4">
          <div v-if="tutor" class="card shadow-sm">
            <div class="card-body p-4">
              <h3 class="h5">Cadastrar novo pet</h3>
              <p class="small text-body-secondary mb-4">
                O pet será vinculado automaticamente a
                {{ tutor.nomeCompleto || tutor.nome }}.
              </p>

              <form @submit.prevent="salvarPet">
                <div class="mb-3">
                  <label for="nome-pet" class="form-label">Nome do pet</label>
                  <input
                    id="nome-pet"
                    v-model="novoPet.nome"
                    type="text"
                    class="form-control"
                    placeholder="Digite o nome do pet"
                    required
                  />
                </div>

                <div class="mb-4">
                  <label for="especie-pet" class="form-label">Espécie</label>
                  <select
                    id="especie-pet"
                    v-model="novoPet.especie"
                    class="form-select"
                    required
                  >
                    <option value="" disabled>Selecione</option>
                    <option value="Cachorro">Cachorro</option>
                    <option value="Gato">Gato</option>
                    <option value="Outro">Outro</option>
                  </select>
                </div>

                <button
                  type="submit"
                  class="btn btn-success w-100"
                  :disabled="!novoPet.nome.trim() || !novoPet.especie || salvandoPet"
                >
                  <span
                    v-if="salvandoPet"
                    class="spinner-border spinner-border-sm me-2"
                    aria-hidden="true"
                  ></span>
                  Cadastrar pet
                </button>
              </form>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>
