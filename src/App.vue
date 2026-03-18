<script setup lang="ts">
import { ref } from 'vue'
import GrapesRenderer from './components/GrapesRenderer.vue'

const sampleGrapesData = ref<any>(null)
const siteId = ref('test-site')
const loadingData = ref(false)
const errorData = ref('')
const apiUrl = import.meta.env.VITE_API_URL

const fetchData = async () => {
  loadingData.value = true
  errorData.value = ''
  try {
    // Llamada al API de Nuxt (asumiendo que corre en el puerto 3000)
    // Usamos la URL completa para evitar problemas de proxy en el demo
    const response = await fetch(`${apiUrl}/api/sites/${siteId.value}/content`)
    if (!response.ok) throw new Error('Error al obtener los datos del sitio')
    const data = await response.json()
    
    // Si tenemos gjson lo usamos, sino usamos html/css (el componente soporta ambos)
    sampleGrapesData.value = data.gjson || { html: data.html, css: data.css }
  } catch (err: any) {
    console.error('Fetch error:', err)
    errorData.value = err.message
  } finally {
    loadingData.value = false
  }
}

// Carga inicial
fetchData()
</script>

<template>
  <main class="app-main">
    <header class="app-header">
      <h1>Vue 3 + GrapesJS Renderer</h1>
      <p>Consultando API: <code>{{ apiUrl }}/api/sites/{{ siteId }}</code></p>
      
      <div class="api-controls">
        <input v-model="siteId" placeholder="ID del sitio" @keyup.enter="fetchData" />
        <button @click="fetchData" :disabled="loadingData">Actualizar</button>
      </div>
    </header>

    <div v-if="loadingData" class="info-msg">Cargando desde el servidor...</div>
    <div v-else-if="errorData" class="error-msg">⚠️ {{ errorData }}</div>

    <!-- Uso del componente -->
    <div v-else class="grapes-view-wrapper">
      <GrapesRenderer :gjson="sampleGrapesData" />
    </div>

  </main>
</template>

<style>
/* Estilos globales de la app */
:root {
  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;
  line-height: 1.5;
  font-weight: 400;
  color-scheme: light dark;
  color: rgba(255, 255, 255, 0.87);
  background-color: #242424;
}

body {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
}

#app {
  width: 100%;
  margin: 0;
  padding: 0;
  text-align: center;
}

.app-main {
  width: 100%;
}

.grapes-view-wrapper {
  width: 100%;
  margin-top: 2rem;
}

.app-header {
  margin-bottom: 3rem;
}

.app-header h1 {
  font-size: 2.5rem;
  margin-bottom: 0.5rem;
}

.app-header p {
  color: #888;
  margin-bottom: 1rem;
}

.api-controls {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-top: 1rem;
}

input {
  padding: 8px 12px;
  border-radius: 6px;
  border: 1px solid #444;
  background: #333;
  color: white;
  width: 200px;
}

button {
  padding: 8px 20px;
  background: #6366f1;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.info-msg, .error-msg {
  padding: 2rem;
  background: #2a2a2a;
  border-radius: 12px;
  margin-top: 1rem;
}

.error-msg {
  color: #ef4444;
  border: 1px solid rgba(239, 68, 68, 0.2);
}
</style>
