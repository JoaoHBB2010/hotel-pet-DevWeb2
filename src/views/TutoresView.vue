<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const tutores = ref([]);
const pets = ref([]);
const erro = ref('');

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

  if (!window.confirm(`Deseja realmente excluir o tutor ${tutor.nome}?`)) {
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
          <th>Nome</th>
          <th>CPF</th>
          <th>Telefone</th>
          <th>E-mail</th>
          <th>Pets(Quantidade)</th>
          <th>Nomes dos pets</th>
          <th>Ações</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="tutor in tutores" :key="tutor.id">
          <td>{{ tutor.id }}</td>
          <td>{{ tutor.nome }}</td>
          <td></td>
          <td>{{ tutor.telefone }}</td>
          <td></td>
          <td></td>
          <td></td>
          <td>
            <button class="btn btn-danger" @click="excluirTutor(tutor)">
              Excluir
            </button>
          </td>
        </tr>
        <tr v-if="tutores.length === 0 && !erro">
          <td colspan="4" class="text-center text-body-secondary">
            Nenhum tutor cadastrado.
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>