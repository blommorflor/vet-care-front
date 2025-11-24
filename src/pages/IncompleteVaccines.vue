<template>
  <div class="pa-4">
    <v-card border>
      <v-card-title class="text-red-darken-2 mb-2 d-flex align-center">
        <v-icon start icon="mdi-alert-circle-outline" class="mr-2"></v-icon>
        Alertas de Vacunación
      </v-card-title>
      <v-card-subtitle class="mb-4">
        Listado de pacientes que requieren dosis o refuerzos según su especie.
      </v-card-subtitle>

      <v-data-table
        :headers="headers"
        :items="items"
        :loading="loading"
        no-data-text="¡Excelente! Todos los pacientes están al día."
      >
        <template #item.nombre="{ item }">
          <div class="text-h6 text-body-1 font-weight-bold">{{ item.nombre }}</div>
          <div class="text-caption text-grey text-capitalize">{{ item.especie }} - {{ item.raza || 'Mestizo' }}</div>
        </template>

        <template #item.dueno="{ item }">
          <div v-if="item.dueno_id">
            <div class="font-weight-medium">{{ item.dueno_id.nombre }}</div>
            <div class="text-caption d-flex align-center mt-1">
               <v-icon size="x-small" start class="mr-1">mdi-phone</v-icon>
               <span v-if="item.dueno_id.contacto">{{ item.dueno_id.contacto.telefono }}</span>
               <span v-else class="text-grey font-italic">Sin teléfono</span>
            </div>
          </div>
          <span v-else class="text-grey">—</span>
        </template>

        <template #item.vacunas_faltantes="{ item }">
          <v-chip
            v-for="(vacuna, index) in item.vacunas_faltantes"
            :key="index"
            color="red-lighten-5"
            text-color="red-darken-4"
            class="ma-1 font-weight-bold border-red-lighten-4"
            label
            size="small"
            variant="flat"
            border
          >
            {{ vacuna }}
          </v-chip>
        </template>
        
        <template #item.vacunas_aplicadas="{ item }">
             <span v-if="item.nombres_vacunas_aplicadas && item.nombres_vacunas_aplicadas.length" class="text-caption text-green-darken-1">
                {{ item.nombres_vacunas_aplicadas.join(', ') }}
             </span>
             <span v-else class="text-caption text-grey">Ninguna</span>
        </template>

      </v-data-table>
    </v-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const items = ref([]);
const loading = ref(false);

const headers = [
  { title: 'Paciente', key: 'nombre', width: '25%' },
  { title: 'Dueño de Contacto', key: 'dueno', width: '25%' },
  { title: 'Dosis Pendientes', key: 'vacunas_faltantes', width: '30%' },
  { title: 'Historial (Aplicadas)', key: 'vacunas_aplicadas' },
];

async function loadReport() {
  loading.value = true;
  try {
    // Llamamos al endpoint que creamos en PetsController
    // Ruta: /api/pets/reports/incomplete-vaccinations
    const res = await axios.get('/api/pets/reports/incomplete-vaccinations');
    items.value = res.data;
  } catch (error) {
    console.error("Error cargando reporte de vacunas:", error);
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadReport();
});
</script>