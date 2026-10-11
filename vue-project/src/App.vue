<template>
  <div class="page">
    <div class="card">
      <header class="card-header">
        <h1>MiniCore</h1>
        <p class="subtitle">Cálculo de comisiones por vendedor</p>
      </header>

      <div class="card-body">
        <div class="form">
          <div class="field">
            <label for="desde">Desde</label>
            <input id="desde" type="date" v-model="fechaInicio" />
          </div>

          <div class="field">
            <label for="hasta">Hasta</label>
            <input id="hasta" type="date" v-model="fechaFin" />
          </div>

          <button @click="calcularComisiones" :disabled="cargando">
            <span v-if="cargando" class="spinner"></span>
            {{ cargando ? 'Calculando...' : 'Calcular' }}
          </button>
        </div>

        <p v-if="error" class="error">{{ error }}</p>

        <div v-if="resultado.length > 0" class="resultado">
          <table>
            <thead>
              <tr>
                <th>Usuario</th>
                <th>Total ventas</th>
                <th>% comisión</th>
                <th>Comisión</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in resultado" :key="r.usuario" :class="{ destacado: r.comision_calculada > 0 }">
                <td class="usuario">{{ r.usuario }}</td>
                <td>${{ r.total_ventas.toFixed(2) }}</td>
                <td>
                  <span class="badge">{{ (r.porcentaje_aplicado * 100).toFixed(0) }}%</span>
                </td>
                <td class="comision">${{ r.comision_calculada.toFixed(2) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const fechaInicio = ref('2025-05-01')
const fechaFin = ref('2025-06-30')
const resultado = ref([])
const cargando = ref(false)
const error = ref('')

async function calcularComisiones() {
  error.value = ''

  if (!fechaInicio.value || !fechaFin.value) {
    error.value = 'Debes seleccionar ambas fechas'
    return
  }

  cargando.value = true
  try {
    const res = await fetch(`/comision?fecha_inicio=${fechaInicio.value}&fecha_fin=${fechaFin.value}`)
    if (!res.ok) throw new Error(`Respuesta del servidor: ${res.status}`)
    const data = await res.json()
    resultado.value = data.resultado
  } catch (err) {
    error.value = 'No se pudo conectar con el servidor'
    console.error(err)
  } finally {
    cargando.value = false
  }
}

calcularComisiones()
</script>

<style>
:root {
  color-scheme: light;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: 'Segoe UI', Roboto, Arial, sans-serif;
  background: linear-gradient(135deg, #6a5bff 0%, #7c3aed 100%);
  min-height: 100vh;
}

.page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem 1rem;
}

.card {
  width: 100%;
  max-width: 760px;
  background: #fff;
  border-radius: 16px;
  box-shadow: 0 20px 50px rgba(31, 12, 75, 0.25);
  overflow: hidden;
}

.card-header {
  background: linear-gradient(135deg, #7c3aed, #a855f7);
  color: #fff;
  padding: 1.75rem 2rem;
}

.card-header h1 {
  margin: 0;
  font-size: 1.75rem;
  font-weight: 700;
  letter-spacing: -0.02em;
}

.subtitle {
  margin: 0.25rem 0 0;
  opacity: 0.9;
  font-size: 0.95rem;
}

.card-body {
  padding: 1.75rem 2rem 2rem;
}

.form {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-end;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.field {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.field label {
  font-size: 0.8rem;
  font-weight: 600;
  color: #555;
  text-transform: uppercase;
  letter-spacing: 0.03em;
}

input[type="date"] {
  padding: 0.55rem 0.75rem;
  border: 1.5px solid #e1e1e8;
  border-radius: 8px;
  font-size: 0.95rem;
  color: #2a2a2a;
  background: #fafafc;
  transition: border-color 0.15s ease;
}

input[type="date"]:focus {
  outline: none;
  border-color: #7c3aed;
  background: #fff;
}

button {
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 1.4rem;
  border: none;
  border-radius: 8px;
  background: #7c3aed;
  color: #fff;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s ease, transform 0.1s ease;
}

button:hover:not(:disabled) {
  background: #6d28d9;
}

button:active:not(:disabled) {
  transform: scale(0.98);
}

button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255, 255, 255, 0.5);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error {
  background: #fef2f2;
  color: #b91c1c;
  padding: 0.6rem 0.9rem;
  border-radius: 8px;
  font-size: 0.9rem;
  margin-bottom: 1rem;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
}

thead th {
  text-align: left;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: #888;
  padding: 0.5rem 0.75rem;
  border-bottom: 2px solid #eee;
}

tbody td {
  padding: 0.75rem;
  border-bottom: 1px solid #f0f0f3;
}

tbody tr:last-child td {
  border-bottom: none;
}

tbody tr:hover {
  background: #faf8ff;
}

.usuario {
  font-weight: 600;
  text-transform: capitalize;
  color: #2a2a2a;
}

.badge {
  display: inline-block;
  padding: 0.15rem 0.55rem;
  border-radius: 999px;
  background: #f1e9ff;
  color: #7c3aed;
  font-weight: 600;
  font-size: 0.85rem;
}

.comision {
  font-weight: 700;
  color: #16a34a;
}

tr.destacado .comision {
  color: #15803d;
}

@media (max-width: 480px) {
  .card-header, .card-body {
    padding: 1.25rem;
  }
  .form {
    flex-direction: column;
    align-items: stretch;
  }
  button {
    justify-content: center;
  }
}
</style>
