<script setup>
import { computed, ref } from 'vue';
import { RouterLink, useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const router = useRouter();

const novoTutor = ref({
  nome: '',
  telefone: '',
  cpfCnpj: '',
  email: '',
  dataNascimento: '',
  endereco: {
    rua: '',
    numero: '',
    bairro: '',
    cidade: '',
    estado: '',
    cep: '',
  },
});
const erro = ref('');
const salvando = ref(false);

const estados = [
  ['AC', 'Acre'],
  ['AL', 'Alagoas'],
  ['AP', 'Amapá'],
  ['AM', 'Amazonas'],
  ['BA', 'Bahia'],
  ['CE', 'Ceará'],
  ['DF', 'Distrito Federal'],
  ['ES', 'Espírito Santo'],
  ['GO', 'Goiás'],
  ['MA', 'Maranhão'],
  ['MT', 'Mato Grosso'],
  ['MS', 'Mato Grosso do Sul'],
  ['MG', 'Minas Gerais'],
  ['PA', 'Pará'],
  ['PB', 'Paraíba'],
  ['PR', 'Paraná'],
  ['PE', 'Pernambuco'],
  ['PI', 'Piauí'],
  ['RJ', 'Rio de Janeiro'],
  ['RN', 'Rio Grande do Norte'],
  ['RS', 'Rio Grande do Sul'],
  ['RO', 'Rondônia'],
  ['RR', 'Roraima'],
  ['SC', 'Santa Catarina'],
  ['SP', 'São Paulo'],
  ['SE', 'Sergipe'],
  ['TO', 'Tocantins'],
];

const telefoneFormatado = computed({
  get() {
    const digitos = novoTutor.value.telefone;
    if (digitos.length <= 2) {
      return digitos;
    }

    const numero = digitos.slice(2);
    const tamanhoInicial = digitos.length > 10 ? 5 : 4;
    return `(${digitos.slice(0, 2)}) ${numero.slice(0, tamanhoInicial)}${
      numero.length > tamanhoInicial ? `-${numero.slice(tamanhoInicial)}` : ''
    }`;
  },
  set(valor) {
    novoTutor.value.telefone = valor.replace(/\D/g, '').slice(0, 11);
  },
});

const cpfCnpjFormatado = computed({
  get() {
    const digitos = novoTutor.value.cpfCnpj;
    if (digitos.length <= 11) {
      return digitos
        .replace(/(\d{3})(\d)/, '$1.$2')
        .replace(/(\d{3})(\d)/, '$1.$2')
        .replace(/(\d{3})(\d{1,2})$/, '$1-$2');
    }

    return digitos
      .replace(/(\d{2})(\d)/, '$1.$2')
      .replace(/(\d{3})(\d)/, '$1.$2')
      .replace(/(\d{3})(\d)/, '$1/$2')
      .replace(/(\d{4})(\d{1,2})$/, '$1-$2');
  },
  set(valor) {
    novoTutor.value.cpfCnpj = valor.replace(/\D/g, '').slice(0, 14);
  },
});

const cepFormatado = computed({
  get() {
    return novoTutor.value.endereco.cep.replace(/(\d{5})(\d)/, '$1-$2');
  },
  set(valor) {
    novoTutor.value.endereco.cep = valor.replace(/\D/g, '').slice(0, 8);
  },
});

const nomeValido = computed(() => novoTutor.value.nome.trim().length > 0);
const telefoneValido = computed(() => novoTutor.value.telefone.length > 0);
const formularioValido = computed(() => nomeValido.value && telefoneValido.value);

async function salvarTutor() {
  if (!formularioValido.value || salvando.value) {
    return;
  }

  salvando.value = true;
  erro.value = '';

  try {
    const resposta = await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nome: novoTutor.value.nome.trim(),
        telefone: novoTutor.value.telefone,
        cpfCnpj: novoTutor.value.cpfCnpj,
        email: novoTutor.value.email.trim(),
        dataNascimento: novoTutor.value.dataNascimento,
        endereco: {
          rua: novoTutor.value.endereco.rua.trim(),
          numero: novoTutor.value.endereco.numero.trim(),
          bairro: novoTutor.value.endereco.bairro.trim(),
          cidade: novoTutor.value.endereco.cidade.trim(),
          estado: novoTutor.value.endereco.estado,
          cep: novoTutor.value.endereco.cep,
        },
      }),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível salvar o tutor.');
    }

    await router.push('/tutores');
  } catch (error) {
    console.error('Erro ao salvar tutor:', error);
    erro.value = 'Não foi possível salvar o tutor. Tente novamente.';
  } finally {
    salvando.value = false;
  }
}
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Cadastro de Tutor</h1>
      <p class="text-body-secondary mb-0">Cadastre os dados do tutor no sistema.</p>
    </header>

    <div v-if="erro" class="alert alert-danger" role="alert">
      {{ erro }}
    </div>

    <form @submit.prevent="salvarTutor">
      <section class="card shadow-sm mb-4" aria-labelledby="dados-tutor">
        <div class="card-body p-4">
          <h2 id="dados-tutor" class="h5 mb-4">Dados do Tutor</h2>
          <div class="row g-3">
            <div class="col-12 col-md-6">
              <label for="nome" class="form-label">Nome completo:</label>
              <input
                id="nome"
                v-model="novoTutor.nome"
                type="text"
                class="form-control"
                autocomplete="name"
                required
                :aria-invalid="!nomeValido"
              />
              <small v-if="!nomeValido" class="text-danger">
                Ops, campo obrigatório, por favor preencha-o!
              </small>
            </div>

            <div class="col-12 col-md-6">
              <label for="telefone" class="form-label">Telefone:</label>
              <input
                id="telefone"
                v-model="telefoneFormatado"
                type="tel"
                class="form-control"
                inputmode="numeric"
                autocomplete="tel"
                placeholder="(47) 99999-0001"
                required
                :aria-invalid="!telefoneValido"
              />
              <small v-if="!telefoneValido" class="text-danger">
                Ops, campo obrigatório, por favor preencha-o!
              </small>
            </div>

            <div class="col-12 col-md-6">
              <label for="cpf-cnpj" class="form-label">CPF/CNPJ:</label>
              <input
                id="cpf-cnpj"
                v-model="cpfCnpjFormatado"
                type="text"
                class="form-control"
                inputmode="numeric"
                autocomplete="off"
                placeholder="Digite o CPF ou CNPJ"
              />
            </div>

            <div class="col-12 col-md-6">
              <label for="email" class="form-label">E-mail:</label>
              <input
                id="email"
                v-model="novoTutor.email"
                type="email"
                class="form-control"
                autocomplete="email"
                placeholder="nome@exemplo.com"
              />
            </div>

            <div class="col-12 col-md-6">
              <label for="data-nascimento" class="form-label">
                Data de nascimento:
              </label>
              <input
                id="data-nascimento"
                v-model="novoTutor.dataNascimento"
                type="date"
                class="form-control"
                autocomplete="bday"
              />
            </div>
          </div>
        </div>
      </section>

      <section class="card shadow-sm mb-4" aria-labelledby="endereco-tutor">
        <div class="card-body p-4">
          <h2 id="endereco-tutor" class="h5 mb-4">Endereço</h2>
          <div class="row g-3">
            <div class="col-12 col-md-8">
              <label for="rua" class="form-label">Rua:</label>
              <input
                id="rua"
                v-model="novoTutor.endereco.rua"
                type="text"
                class="form-control"
                autocomplete="address-line1"
              />
            </div>

            <div class="col-12 col-md-4">
              <label for="numero" class="form-label">Número:</label>
              <input
                id="numero"
                v-model="novoTutor.endereco.numero"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-12 col-md-6">
              <label for="bairro" class="form-label">Bairro:</label>
              <input
                id="bairro"
                v-model="novoTutor.endereco.bairro"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-12 col-md-6">
              <label for="cidade" class="form-label">Cidade:</label>
              <input
                id="cidade"
                v-model="novoTutor.endereco.cidade"
                type="text"
                class="form-control"
                autocomplete="address-level2"
              />
            </div>

            <div class="col-12 col-md-6">
              <label for="estado" class="form-label">Estado:</label>
              <select
                id="estado"
                v-model="novoTutor.endereco.estado"
                class="form-select"
                autocomplete="address-level1"
              >
                <option value="" disabled>Selecione o estado</option>
                <option
                  v-for="[sigla, nome] in estados"
                  :key="sigla"
                  :value="sigla"
                >
                  {{ nome }} ({{ sigla }})
                </option>
              </select>
            </div>

            <div class="col-12 col-md-6">
              <label for="cep" class="form-label">CEP:</label>
              <input
                id="cep"
                v-model="cepFormatado"
                type="text"
                class="form-control"
                inputmode="numeric"
                autocomplete="postal-code"
                placeholder="00000-000"
              />
            </div>
          </div>
        </div>
      </section>

      <button
        type="submit"
        class="btn btn-success"
        :disabled="!formularioValido || salvando"
      >
        {{ salvando ? 'Salvando...' : 'Salvar tutor' }}
      </button>
      <RouterLink class="btn btn-secondary ms-2" to="/tutores">Cancelar</RouterLink>
    </form>
  </div>
</template>
