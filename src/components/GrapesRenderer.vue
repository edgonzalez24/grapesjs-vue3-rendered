<template>
  <div class="grapes-renderer-container">
    <div v-if="loading" class="loading-state">
      <p>Procesando diseño...</p>
    </div>
    <div v-else class="rendered-content" v-html="output" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'

const props = defineProps<{
  gjson: any
}>()

const output = ref('')
const loading = ref(true)

const renderContent = async () => {
  if (!props.gjson) {
    loading.value = false
    return
  }

  loading.value = true
  
  try {
    // Importación dinámica de GrapesJS
    const grapesjs = (await import('grapesjs')).default
    
    // Div temporal "fuera del DOM"
    const el = document.createElement('div')
    
    const editor = grapesjs.init({
      container: el,
      headless: true, // Esto evita que cree la UI completa
      storageManager: false,
    })

    // Cargar los datos
    editor.loadProjectData(props.gjson)
    
    // Extraer HTML y CSS
    const html = editor.getHtml()
    const css = editor.getCss()
    
    // Combinar en el output
    output.value = `<style>${css}</style><div>${html}</div>`
    
    // Destruir la instancia de memoria
    editor.destroy()
  } catch (error) {
    console.error('Error rendering GrapesJS:', error)
    output.value = '<p>Error al renderizar el contenido.</p>'
  } finally {
    loading.value = false
  }
}

onMounted(renderContent)

// Re-renderizar si el JSON cambia (reactividad)
watch(() => props.gjson, renderContent, { deep: true })
</script>

<style scoped>
.grapes-renderer-container {
  width: 100%;
}
.loading-state {
  padding: 2rem;
  text-align: center;
  color: #666;
}
</style>
