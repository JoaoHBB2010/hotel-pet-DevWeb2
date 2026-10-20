<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const tutores = ref([]);
const pets = ref([]);
const erro = ref('');

function petsDoTutor(tutorId) {
  return pets.value.filter((pet) => String(pet.tutorId) === String(tutorId));
}

function formatarCpfCnpj(valor) {
  if (!valor) {
    return '-';
  }

  if (valor.includes('*')) {
    return valor;
  }

  const digitos = valor.replace(/\D/g, '');
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
    return '-';
  }

  const [ano, mes, dia] = valor.split('-');
  return dia && mes && ano ? `${dia}/${mes}/${ano}` : valor;
}

async function carregarDados() {
  try {
    const [respostaTutores, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores`),
      fetch(`${API_URL}/pets`),
    ]);

    if (!respostaTutores.ok || !respostaPets.ok) {
      throw new Error('Não foi possível carregar os tutores.');
    }

    tutores.value = await respostaTutores.json();
    pets.value = await respostaPets.json();
    erro.value = '';
  } catch (error) {
    console.error('Erro ao carregar tutores:', error);
    erro.value = 'Não foi possível carregar os tutores. Tente novamente.';
  }
}

async function excluirTutor(tutor) {
  if (pets.value.some((pet) => String(pet.tutorId) === String(tutor.id))) {
    window.alert('Não é possível excluir este tutor enquanto houver pets vinculados.');
    return;
  }

  if (!window.confirm('Deseja realmente remover este tutor?')) {
    return;
  }

  try {
    const resposta = await fetch(`${API_URL}/tutores/${tutor.id}`, {
      method: 'DELETE',
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível excluir o tutor.');
    }

    await carregarDados();
  } catch (error) {
    console.error('Erro ao excluir tutor:', error);
    erro.value = 'Não foi possível excluir o tutor. Tente novamente.';
  }
}

onMounted(carregarDados);
</script>

<template>
  <div>
    <header class="mb-4 d-flex flex-wrap justify-content-between align-items-center gap-3">
      <div>
        <h1 class="text-2xl font-bold">Listagem de Tutores</h1>
        <p class="text-body-secondary mb-0">Tutores cadastrados no sistema.</p>
      </div>
      <RouterLink class="btn btn-success" to="/tutores/novo">
        Cadastrar tutor
      </RouterLink>
    </header>

    <div v-if="erro" class="alert alert-danger" role="alert">
      {{ erro }}
    </div>

    <table class="table table-striped table-hover">
      <thead>
        <tr>
          <th>ID</th>
          <th>Nome completo</th>
          <th>CPF/CNPJ</th>
          <th>Telefone</th>
          <th>E-mail</th>
          <th>Data de nascimento</th>
          <th>Quantidade de pets</th>
          <th>Pets vinculados</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="tutor in tutores" :key="tutor.id">
          <td>{{ tutor.id }}</td>
          <td>{{ tutor.nomeCompleto || tutor.nome || '-' }}</td>
          <td>{{ formatarCpfCnpj(tutor.cpfCnpj || tutor.cpf) }}</td>
          <td>{{ formatarTelefone(tutor.telefone) }}</td>
          <td>{{ tutor.email || '-' }}</td>
          <td>{{ formatarData(tutor.dataNascimento || tutor.dataDeNascimento) }}</td>
          <td>{{ petsDoTutor(tutor.id).length }}</td>
          <td>
            {{ petsDoTutor(tutor.id).map((pet) => pet.nome).join(', ') || 'Nenhum pet' }}
          </td>
          <td>
            <RouterLink
              class="btn btn-primary me-2"
              :to="`/tutores/${tutor.id}`"
            >
              Visualizar
            </RouterLink>
            <button class="btn btn-danger" @click="excluirTutor(tutor)">
              Excluir
            </button>
          </td>
        </tr>
        <tr v-if="tutores.length === 0 && !erro">
          <td colspan="10" class="text-center text-body-secondary">
            Nenhum tutor cadastrado.
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>