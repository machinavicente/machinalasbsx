<script setup lang="ts">
import type { SupabaseClient } from '@supabase/supabase-js'

interface simuladores {
  id: number
  nombre_del_simulador: string
  enlace: string
  categoria: string
  asignatura: string
  descripcion_del_simulador: string
}

// Obtén el cliente Supabase con el tipo correcto
const { $supabase } = useNuxtApp()
const supabase = $supabase as SupabaseClient

// Estados reactivos
const simuladores = ref<simuladores[]>([])
const searchTerm = ref('')
const isLoading = ref(true)
const error = ref<string | null>(null)
const asignaturaActual = ref('Todas las asignaturas')

// Función para cargar datos
async function cargarSimuladores() {
  try {
    isLoading.value = true;
    error.value = null;

    // 1. Verificar conexión básica
    const { data: sessionData } = await supabase.auth.getSession();
    console.log('Sesión Supabase:', sessionData);

    // 2. Verificar existencia de la tabla
    const { data: tableInfo } = await supabase
      .rpc('get_table_info', { table_name: 'simuladores' })
      .single();
    console.log('Información de tabla:', tableInfo);

    // 3. Consulta con conteo exacto
    const { data, error: supabaseError, count } = await supabase
      .from('simuladores')
      .select('*', { count: 'exact' })
      .order('nombre_del_simulador', { ascending: true });

    console.log('Respuesta completa:', { data, error: supabaseError, count });

    if (supabaseError) throw supabaseError;

    if (count === 0) {
      console.warn('La tabla existe pero está vacía');
      error.value = 'No se encontraron simuladores en la base de datos';
    }

    simuladores.value = data || [];

  } catch (err) {
    error.value = err instanceof Error ? err.message : 'Error de Conexion, Verifique e intente de nuevo  ';
    console.error('Error completo:', err);
  } finally {
    isLoading.value = false;
  }
}

onMounted(() => {
  cargarSimuladores()
})

const filteredSimuladores = computed(() => {
  if (!simuladores.value) return []

  // Función para normalizar el texto (quitar acentos y caracteres especiales)
  const normalizeText = (text) => {
    return text.toLowerCase()
      .normalize('NFD') // Descompone los caracteres acentuados
      .replace(/[\u0300-\u036f]/g, '') // Elimina los diacríticos
      .trim()
  }

  const term = normalizeText(searchTerm.value)

  return term
    ? simuladores.value.filter(sim =>
      normalizeText(sim.nombre_del_simulador).includes(term) ||
      normalizeText(sim.categoria).includes(term))
    : simuladores.value
})
</script>
<template>
  <div class="contendor">
    
  </div>
  <!-- Barra de búsqueda -->
  <div class="search-container text-white">
    <div class="container">
      <div class="search-box">
        <h2 class="text-center mb-4">Encuentra el simulador que necesitas</h2>
        <div class="input-group mb-3">
          <input v-model="searchTerm" type="text" class="search-input" placeholder="Buscar simuladores...">
        </div>
        <div class="text-center">
          <small>Ejemplo: redes, programación, bases de datos</small>
        </div>
      </div>
    </div>
  </div>

  <!-- Sección por asignatura -->
  <div class="container  containerr py-4">
    <h3 class="section-title mb-4">{{ asignaturaActual }}</h3>

    <div v-if="isLoading" class="text-center py-5">
      <div class="spinner-border text-primary" role="status">
        <span class="visually-hidden">Cargando...</span>
      </div>
    </div>

    <div v-else-if="error" class="alert alert-danger">
      {{ error }}
    </div>

    <div v-else class="row g-4 mb-5">
      <!-- Card Individual -->
      <div v-for="sim in filteredSimuladores" :key="sim.id" class="col-md-4 col-lg-3">
        <div class="card shadow-sm h-100">
          <div class="card-body text-center d-flex flex-column">
            <i class="bi bi-terminal fs-1 my-4"></i>
            <span class="badge category-badge mb-3">{{ sim.categoria }}</span>
            <h5 class="card-title">{{ sim.nombre_del_simulador }}</h5>
            <hr>
            <p class="card-text text-muted mb-4">{{ sim.descripcion_del_simulador }}</p>
            <a :href="sim.enlace" target="_blank" class="btn btn-simulator mt-auto btn-primary">
              Ejecutar
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped src="/public/assets/css/home.css">
</style>