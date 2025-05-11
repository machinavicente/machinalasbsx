<template>
  <div class="container py-5">
    <div class="row justify-content-center">
      <div class="col-lg-8">
        <div class="card shadow-lg border-0">
          <div class="card-header unefa-primary-bg text-white">
            <h3 class="mb-0">
              <i class="bi bi-plus-square me-2"></i>Registrar Nuevo Simulador
            </h3>
          </div>
          <div class="card-body">
            <!-- Notificaciones -->
            <div v-if="successMessage" class="alert alert-success alert-dismissible fade show">
              {{ successMessage }}
              <button type="button" class="btn-close" @click="successMessage = null"></button>
            </div>
            <div v-if="error" class="alert alert-danger alert-dismissible fade show">
              {{ error }}
              <button type="button" class="btn-close" @click="error = null"></button>
            </div>

            <form @submit.prevent="submitForm">
              <div class="row g-3">
                <!-- Nombre del Simulador -->
                <div class="col-md-12">
                  <label for="nombre" class="form-label">Nombre del Simulador: </label>
                  <input type="text" class="form-control" id="nombre" v-model="simulador.nombre_del_simulador" required
                    placeholder="SQLMaster - Simulador de Bases de Datos">
                </div>

                <!-- Descripción -->
                <div class="col-md-12">
                  <label for="descripcion" class="form-label">Descripción: </label>
                  <textarea class="form-control" id="descripcion" rows="3" v-model="simulador.descripcion_del_simulador"
                    required
                    placeholder="Entorno seguro para practicar consultas complejas, optimización y diseño de esquemas relacionales"></textarea>
                </div>

                <!-- Categoría y Enlace -->
                <div class="col-md-6">
                  <label for="asignatura" class="form-label">Asignatura: </label>
                  <select class="form-select" id="categoria" v-model="simulador.asignatura" required>
                    <option value="" disabled>Seleccionar...</option>
                    <option value="Matemáticas">Matemáticas</option>
                    <option value="Quimica">Quimica</option>
                    <option value="Física I">Física I</option>
                    <option value="Física II">Física II</option>
                    <option value="Programación">Programación</option>
                    <option value="Lenguajes de Programacion I">Lenguajes de Programación I</option>
                    <option value="Lenguajes de Programacion II">Lenguajes de Programación II</option>
                    <option value="Lenguajes de Programacion III">Lenguajes de Programación III</option>
                    <option value="Procesamiento de Datos">Procesamiento de Datos</option>
                    <option value="Bases de Datos">Bases de Datos</option>
                    <option value="Redes">Redes</option>
                    <option value="Sistemas Operativos">Sistemas Operativos</option>
                    <option value="Simulacion y Modelos">Simulación y Modelos</option>
                  </select>
                </div>

                <div class="col-md-6">
                  <label for="enlace" class="form-label">Enlace del Simulador: </label>
                  <input type="url" class="form-control" id="enlace" v-model="simulador.enlace" required
                    placeholder="https://www.ejemplo.com">
                </div>

                <!-- Asignatura -->
                <div class="col-md-6">
                  <label for="categoria" class="form-label">Categoria: </label>
                  <input type="text" class="form-control" id="categoria" v-model="simulador.categoria"
                    placeholder="Ej: Compilador, Diagramador, IDE..">
                </div>
                <div class="col-md-6">
                  <div class="col-md-6">
                    <label for="fecha" class="form-label">Fecha de Registro: </label>
                    <input type="datetime-local" class="form-control" id="fecha" v-model="simulador.created_at"
                      readonly />
                  </div>
                </div>

                <!-- Botones de acción -->
                <div class="col-12 mt-4">
                  <div class="d-flex justify-content-between">
                    <button type="button" class="btn btn-outline-danger" @click="resetForm">
                      <i class="bi bi-x-circle me-2"></i>Cancelar
                    </button>

                    <button type="submit" class="btn-unefa btn" :disabled="loading">
                      <template v-if="loading">
                        <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>
                        Procesando...
                      </template>
                      <template v-else>
                        <i class="bi bi-save me-2"></i>Guardar Simulador
                      </template>
                    </button>
                  </div>
                </div>
              </div>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { createClient } from '@supabase/supabase-js'

export default {
  data() {
    const now = new Date()
    const venezuelaOffset = -4 * 60 * 60 * 1000
    const venezuelaTime = new Date(now.getTime() + venezuelaOffset)
    const formattedNow = venezuelaTime.toISOString().slice(0, 16)

    return {
      simulador: {
        nombre_del_simulador: '',
        descripcion_del_simulador: '',
        categoria: '',
        enlace: '',
        asignatura: '',
        created_at: formattedNow
      },
      loading: false,
      error: null,
      successMessage: null,
      supabase: null
    }
  },
  created() {
    this.initializeSupabase()
  },
  methods: {
    initializeSupabase() {
      const supabaseUrl = 'https://nthgofwioyfrjvocyvrs.supabase.co'
      const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im50aGdvZndpb3lmcmp2b2N5dnJzIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDU1NDcyOTQsImV4cCI6MjA2MTEyMzI5NH0.J80rYjQg6NWbo_yNNxcVnTbPavKmpxwZIi5UNrBTG84'
      this.supabase = createClient(supabaseUrl, supabaseKey)
    },
    async submitForm() {
      if (!this.validateForm()) return

      this.loading = true
      this.error = null
      this.successMessage = null

      try {
        const now = new Date()
        const venezuelaOffset = -4 * 60 * 60 * 1000
        const venezuelaTime = new Date(now.getTime() + venezuelaOffset)

        const { data, error } = await this.supabase
          .from('simuladores')
          .insert([
            {
              nombre_del_simulador: this.simulador.nombre_del_simulador,
              descripcion_del_simulador: this.simulador.descripcion_del_simulador,
              categoria: this.simulador.categoria,
              enlace: this.simulador.enlace,
              asignatura: this.simulador.asignatura,
              created_at: venezuelaTime.toISOString()
            }
          ])
          .select()

        if (error) throw error

        console.log('Datos insertados:', data)
        this.showSuccess('Simulador registrado con éxito!')
        this.resetForm()
      } catch (error) {
        console.error('Error al insertar datos:', error)
        this.error = error.message || 'Ocurrió un error al registrar el simulador'
      } finally {
        this.loading = false
      }
    },
    resetForm() {
      const now = new Date()
      const venezuelaOffset = -4 * 60 * 60 * 1000
      const venezuelaTime = new Date(now.getTime() + venezuelaOffset)
      const formattedNow = venezuelaTime.toISOString().slice(0, 16)

      this.simulador = {
        nombre_del_simulador: '',
        descripcion_del_simulador: '',
        categoria: '',
        enlace: '',
        asignatura: '',
        created_at: formattedNow
      }
    },
    validateForm() {
      const forbiddenKeywords = [
        'porn', 'xxx', 'sex', 'facebook', 'tiktok', 'instagram',
        'juegos', 'game', 'playstation', 'xvideos', 'xnxx', 'y8',"porn", "xxx", "sex", "xvideos", "xnxx", "redtube", "youporn", "brazzers", "youjizz", "pornhub", "hentai", "erotic", "adult", "nsfw", "camgirl", "cam4", "onlyfans", "fap", "nude", "escort", "livejasmin", "stripchat", "badoink", "lust", "bangbros", "tushy", "nudity", "climax", "anal",  "facebook", "instagram", "tiktok", "snapchat", "twitter", "x.com", "whatsapp", "messenger", "telegram", "reddit", "pinterest", "linkedin", "discord", "wechat", "line.me", "threads", "game", "games", "gamer", "juego", "juegos", "roblox", "fortnite", "minecraft", "steam", "epicgames", "valorant", "leagueoflegends", "lol", "riotgames", "playstation", "xbox", "nintendo", "pokemon", "callofduty", "counterstrike", "csgo", "dota", "pubg", "freefire", "garena", "twitch"
        
      ]

      const fieldsToCheck = [
        this.simulador.enlace,
        this.simulador.nombre_del_simulador,
        this.simulador.descripcion_del_simulador
      ]

      for (const field of fieldsToCheck) {
        if (!field) continue
        for (const keyword of forbiddenKeywords) {
          if (field.toLowerCase().includes(keyword)) {
            this.error = 'Este tipo de sitio no puede ser registrado.'
            return false
          }
        }
      }

      if (!this.simulador.nombre_del_simulador) {
        this.error = 'El nombre del simulador es requerido'
        return false
      }

      if (!this.simulador.enlace || !this.isValidUrl(this.simulador.enlace)) {
        this.error = 'Por favor ingrese un enlace válido'
        return false
      }

      if (!this.simulador.categoria) {
        this.error = 'Debe seleccionar una categoría'
        return false
      }

      return true
    },
    isValidUrl(string) {
      try {
        new URL(string)
        return true
      } catch (_) {
        return false
      }
    },
    showSuccess(message) {
      this.successMessage = message
      setTimeout(() => {
        this.successMessage = null
      }, 5000)
    }
  }
}
</script>

<style scoped>
.card {
  border-radius: 10px;
  overflow: hidden;
}

.unefa-primary-bg {
  background-color: #003366;
}

.btn-unefa {
  background-color: rgb(34, 197, 74);
  color: white;
  border: none;
}

.btn-unefa:hover {
  background-color: rgb(22, 163, 74);
}

.form-control:focus,
.form-select:focus {
  border-color: #FFC72C;
  box-shadow: 0 0 0 0.25rem rgba(0, 51, 102, 0.25);
}

.alert {
  position: relative;
  padding: 1rem 1rem;
  margin-bottom: 1rem;
  border: 1px solid transparent;
  border-radius: 0.25rem;
}

.alert-success {
  color: #0f5132;
  background-color: #d1e7dd;
  border-color: #badbcc;
}

.alert-danger {
  color: #842029;
  background-color: #f8d7da;
  border-color: #f5c2c7;
}

.btn-close {
  position: absolute;
  top: 0;
  right: 0;
  padding: 1.25rem 1rem;
}
</style>
