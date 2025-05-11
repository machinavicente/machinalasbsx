<template>
  <ClientOnly>
    <div class="dashboard-container unefa-primary-bg text-white">
      <!-- Header del Dashboard -->
      <header class="dashboard-header py-3">
        <div class="container">
          <div class="row align-items-center text-center text-md-start">
            <!-- Logo + Título -->
            <div
              class="col-12 col-md-8 d-flex flex-column flex-md-row align-items-center justify-content-center justify-content-md-start mb-3 mb-md-0">
              <img src="/assets/images/logo.png" alt="logo" width="50" height="70">
              <h1 class="ms-md-3 mt-3 mt-md-0 mb-0">Dashboard
              </h1>
            </div>

            <!-- Fecha y hora -->
            <div class="col-12 col-md-4">
              <div class="current-time">
                <div class="date">{{ currentDate }}</div>
                <div class="time">{{ currentTime }}</div>
              </div>
            </div>
          </div>
        </div>
      </header>

      <!-- Contenido principal del Dashboard -->
      <main class="dashboard-main">
        <div class="container">
          <!-- Sección de métricas -->
          <section class="metrics-section mb-5">
            <div class="row g-4">
              <!-- Total de Simuladores -->
              <div class="col-md-6 col-lg-3">
                <div class="metric-card">
                  <div class="metric-icon">
                    <i class="bi bi-cpu"></i>
                  </div>
                  <div class="metric-content">
                    <h3>Total de Simuladores</h3>
                    <p class="metric-value">{{ totalSimuladores }}</p>
                  </div>
                </div>
              </div>

              <!-- Total de Asignaturas -->
              <div class="col-md-6 col-lg-3">
                <div class="metric-card">
                  <div class="metric-icon">
                    <i class="bi bi-journal-bookmark"></i>
                  </div>
                  <div class="metric-content">
                    <h3>Total de Asignaturas</h3>
                    <p class="metric-value">{{ totalAsignaturas }}</p>
                  </div>
                </div>
              </div>

              <!-- Simuladores Recientes -->
              <div class="col-md-6 col-lg-3">
                <div class="metric-card">
                  <div class="metric-icon">
                    <i class="bi bi-clock-history"></i>
                  </div>
                  <div class="metric-content">
                    <h3>Simuladores Recientes</h3>
                    <p class="metric-value">{{ simuladoresRecientes.length }}</p>
                  </div>
                </div>
              </div>

              <!-- Asignaturas con Simuladores -->
              <div class="col-md-6 col-lg-3">
                <div class="metric-card">
                  <div class="metric-icon">
                    <i class="bi bi-list-check"></i>
                  </div>
                  <div class="metric-content">
                    <h3>Asignaturas con Simuladores</h3>
                    <p class="metric-value">{{ asignaturasConSimuladores.length }}</p>
                  </div>
                </div>
              </div>
            </div>
          </section>

          <!-- Sección de los 3 simuladores más recientes -->
          <section class="ultimos-simuladores-section mb-5">
            <h2 class="section-title mb-4">
              <i class="bi bi-star-fill me-2"></i>Últimos Simuladores Agregados
            </h2>

            <div v-if="isLoading" class="text-center py-5">
              <div class="spinner-border text-primary" role="status">
                <span class="visually-hidden">Cargando...</span>
              </div>
            </div>

            <div v-else-if="error" class="alert alert-danger">
              {{ error }}
            </div>

            <div v-else class="row g-4">
              <div v-for="sim in ultimosSimuladores" :key="sim.id" class="col-md-4">
                <div class="ultimo-simulador-card">
                  <div class="card-header">
                    <span class="badge bg-warning text-dark">Nuevo</span>
                    <span class="badge category-badge">{{ sim.categoria }}</span>
                  </div>
                  <div class="card-body">
                    <h5>{{ sim.nombre_del_simulador }}</h5>
                    <p class="small">{{ sim.descripcion_del_simulador }}</p>
                    <div class="d-flex justify-content-between align-items-center">
                      <span class="badge asignatura-badge">{{ sim.asignatura }}</span>
                      <a :href="sim.enlace" target="_blank" class="btn btn-sm btn-primary">
                        <i class="bi bi-play-fill"></i> Usar
                      </a>
                      
                    </div>
                    <span class="small text-success">Agregado: {{ formatDate(sim.created_at) }}</span>
                  </div>
                </div>
              </div>
            </div>
          </section>

          <!-- Sección de Acciones -->
          <section class="actions-section">
            <div class="row">
              <div class="col-md-6">
                <div class="action-card download-card">
                  <div class="action-icon">
                    <i class="bi bi-file-earmark-arrow-down"></i>
                  </div>
                  <div class="action-content">
                    <h3>Manual de Usuario</h3>
                    <p>Descarga la guía completa para usar los simuladores</p>
                    <button class="btn btn-download" @click="descargarManual">
                      <i class="bi bi-download me-2"></i> Descargar
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>
      </main>
    </div>
  </ClientOnly>
</template>

<script setup lang="ts">
import type { SupabaseClient } from '@supabase/supabase-js'

interface Simulador {
  id: number
  nombre_del_simulador: string
  enlace: string
  categoria: string
  asignatura: string
  descripcion_del_simulador: string
  created_at: string
}

const { $supabase } = useNuxtApp()
const supabase = $supabase as SupabaseClient

// Datos del dashboard
const simuladores = ref<Simulador[]>([])
const ultimosSimuladores = ref<Simulador[]>([])
const totalSimuladores = ref(0)
const totalAsignaturas = ref(0)
const isLoading = ref(true)
const error = ref<string | null>(null)
const searchTerm = ref('')

// Fecha y hora actual
const currentDate = ref('')
const currentTime = ref('')
const timeInterval = ref<NodeJS.Timeout>()

// Cargar datos
async function cargarDatosDashboard() {
  try {
    isLoading.value = true
    error.value = null

    // 1. Contar total de simuladores
    const { count: countSimuladores, error: countError } = await supabase
      .from('simuladores')
      .select('*', { count: 'exact', head: true })

    if (countError) throw countError
    totalSimuladores.value = countSimuladores || 0

    // 2. Obtener asignaturas únicas
    const { data: asignaturasData, error: asignaturasError } = await supabase
      .from('simuladores')
      .select('asignatura')
      .not('asignatura', 'is', null)

    if (asignaturasError) throw asignaturasError

    // Calcular asignaturas únicas
    const asignaturasUnicas = new Set<string>()
    asignaturasData?.forEach(item => {
      if (item.asignatura) {
        asignaturasUnicas.add(item.asignatura)
      }
    })
    totalAsignaturas.value = asignaturasUnicas.size

    // 3. Cargar los 3 simuladores más recientes
    const { data: ultimosTresData, error: ultimosTresError } = await supabase
      .from('simuladores')
      .select('*')
      .order('created_at', { ascending: false })
      .limit(3)

    if (ultimosTresError) throw ultimosTresError
    ultimosSimuladores.value = ultimosTresData || []

    // 4. Cargar todos los simuladores para el listado completo
    const { data: simuladoresData, error: simuladoresError } = await supabase
      .from('simuladores')
      .select('*')
      .order('nombre_del_simulador', { ascending: true })

    if (simuladoresError) throw simuladoresError
    simuladores.value = simuladoresData || []

  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Error de Conexion, Verifique e intente de nuevo'
    console.error('Error al cargar datos del dashboard:', err)
  } finally {
    isLoading.value = false
  }
}

// Computed properties
const asignaturasConSimuladores = computed(() => {
  const asignaturas = new Set<string>()
  simuladores.value.forEach(sim => {
    if (sim.asignatura) {
      asignaturas.add(sim.asignatura)
    }
  })
  return Array.from(asignaturas)
})

const simuladoresRecientes = computed(() => {
  return simuladores.value.slice(0, 6)
})

const filteredSimuladores = computed(() => {
  if (!searchTerm.value) return simuladores.value

  const term = searchTerm.value.toLowerCase()
  return simuladores.value.filter(sim =>
    sim.nombre_del_simulador.toLowerCase().includes(term) ||
    sim.categoria.toLowerCase().includes(term) ||
    sim.asignatura.toLowerCase().includes(term)
  )
})

// Funciones auxiliares
function countSimuladoresPorAsignatura(asignatura: string) {
  return simuladores.value.filter(sim => sim.asignatura === asignatura).length
}

function formatDate(dateString: string) {
  const options: Intl.DateTimeFormatOptions = { year: 'numeric', month: 'short', day: 'numeric' }
  return new Date(dateString).toLocaleDateString('es-ES', options)
}

function updateDateTime() {
  const now = new Date()
  currentDate.value = now.toLocaleDateString('es-ES', {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
  
  // Formato de 12 horas con AM/PM
  currentTime.value = now.toLocaleTimeString('es-ES', { 
    hour: '2-digit', 
    minute: '2-digit',
    hour12: true
  })
}
function descargarManual() {
  alert('Descargando manual de usuario...')
  // window.open('/path/to/manual.pdf', '_blank')
}

// Inicialización
onMounted(() => {
  cargarDatosDashboard()
  updateDateTime()
  timeInterval.value = setInterval(updateDateTime, 60000)
})

onBeforeUnmount(() => {
  if (timeInterval.value) {
    clearInterval(timeInterval.value)
  }
})
</script>

<style scoped>
.dashboard-container {
  min-height: 100vh;
  border-bottom: 4px solid #C8102E;
}

.dashboard-header {
  background-color: rgba(0, 0, 0, 0.1);
}

.dashboard-main {
  padding: 2rem 0;
}

.current-time {
  text-align: right;
  background: rgba(255, 255, 255, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
}

.current-time .date {
  font-size: 0.9rem;
  opacity: 0.8;
}

.current-time .time {
  font-size: 1.5rem;
  font-weight: bold;
}

/* Tarjetas de métricas */
.metric-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 0.5rem;
  padding: 1.5rem;
  display: flex;
  align-items: center;
  height: 100%;
  transition: transform 0.3s ease, background 0.3s ease;
}

.metric-card:hover {
  background: rgba(255, 255, 255, 0.15);
  transform: translateY(-5px);
}

.metric-icon {
  font-size: 2rem;
  color: #FFC72C;
  margin-right: 1.5rem;
}

.metric-content h3 {
  font-size: 1rem;
  margin-bottom: 0.5rem;
  opacity: 0.8;
}

.metric-value {
  font-size: 1.75rem;
  font-weight: bold;
  margin-bottom: 0;
}

/* Sección de últimos simuladores */
.ultimos-simuladores-section {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 0.5rem;
  padding: 1.5rem;
}

.ultimo-simulador-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 0.5rem;
  overflow: hidden;
  transition: transform 0.3s ease;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.ultimo-simulador-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
}

.ultimo-simulador-card .card-header {
  display: flex;
  justify-content: space-between;
  padding: 0.75rem 1rem;
  background: rgba(0, 0, 0, 0.2);
}

.ultimo-simulador-card .card-body {
  padding: 1.5rem;
  flex-grow: 1;
}

.ultimo-simulador-card .card-footer {
  padding: 0.75rem 1rem;
  background: rgba(0, 0, 0, 0.1);
}

/* Estilos para los simuladores normales */
.simuladores-section .card {
  background: rgba(255, 255, 255, 0.05);
  border: none;
  transition: transform 0.3s ease;
}

.simuladores-section .card:hover {
  transform: translateY(-5px);
  background: rgba(255, 255, 255, 0.1);
}

/* Badges */
.bg-warning {
  background-color: #FFC72C !important;
}

.category-badge {
  background-color: rgba(91, 124, 53, 0.986);
}

.asignatura-badge {
  background-color: rgba(200, 16, 46, 0.8);
}

/* Tarjetas de acciones */
.action-card {
  display: flex;
  border-radius: 0.5rem;
  overflow: hidden;
  height: 100%;
}

.download-card {
  background: rgba(255, 255, 255, 0.05);
}

.stats-card {
  background: rgba(0, 0, 0, 0.1);
}

.action-icon {
  background: rgba(255, 199, 44, 0.2);
  color: #FFC72C;
  font-size: 1.5rem;
  padding: 1.5rem;
  display: flex;
  align-items: center;
}

.action-content {
  padding: 1.5rem;
  flex-grow: 1;
}

.action-content h3 {
  margin-bottom: 0.5rem;
}

.action-content p {
  opacity: 0.8;
  margin-bottom: 1rem;
  font-size: 0.9rem;
}

.btn-download {
  background-color: #C8102E;
  color: white;
}

.btn-stats {
  background-color: #002147;
  color: white;
  opacity: 0.7;
  cursor: not-allowed;
}

.section-title {
  font-size: 1.5rem;
  font-weight: 600;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid rgba(255, 255, 255, 0.1);
}

.btn-simulator {
  background-color: rgba(200, 16, 46, 0.8);
  color: white;
  border: none;
}

.btn-simulator:hover {
  background-color: rgba(200, 16, 46, 1);
}

/* Spinner de carga */
.spinner-border {
  width: 3rem;
  height: 3rem;
}
</style>