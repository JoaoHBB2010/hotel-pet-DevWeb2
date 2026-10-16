<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const router = useRouter();

const novoTutor = ref({
  nome: '',
  telefone: '',
});
const erro = ref('');

async function salvarTutor() {
  try {
    const resposta = await fetch(`${API_URL}/tutores`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(novoTutor.value),
    });

    if (!resposta.ok) {
      throw new Error('Não foi possível salvar o tutor.');
    }

    await router.push('/tutores');
  } catch (error) {
    console.error('Erro ao salvar tutor:', error);
    erro.value = 'Não foi possível salvar o tutor. Tente novamente.';
  }
}
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Cadastro de Tutores</h1>
      <p class="text-body-secondary mb-0">Cadastre um novo tutor no sistema.</p>
    </header>

    <div v-if="erro" class="alert alert-danger" role="alert">
      {{ erro }}
    </div>

    <form @submit.prevent="salvarTutor">
      <div class="col-md-6 mb-3">
        <label for="nome" class="form-label">Nome do tutor:</label>
        <input
          id="nome"
          v-model="novoTutor.nome"
          type="text"
          class="form-control"
          required
        />
      </div>

      <div class="col-md-6 mb-3">
        <label for="telefone" class="form-label">Telefone:</label>
        <input
          id="telefone"
          v-model="novoTutor.telefone"
          type="tel"
          class="form-control"
          required
        />
      </div>

      <div class="col-md-6 mb-3">
        <label for="cpf" class="form-label">CPF/CNPJ :</label>
        <input
          id="cpf"
          v-model="novoTutor.cpf"
          type="Number"
          class="form-control"
          required
          placeholder="Digite o CPF ou CNPJ"
        />
      </div>
      <div class="col-md-6 mb-3">
        <label for="email" class="form-label">Email:</label>
        <input
          id="email"
          v-model="novoTutor.email"
          type="text"
          class="form-control"
          required
          placeholder="nome@examplo.com"
        />
      </div>

      
      <button type="submit" class="btn btn-success">Salvar tutor</button>
    </form>
  </div>
</template>
