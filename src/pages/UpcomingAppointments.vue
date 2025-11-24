<template>
  <div class="pa-4">
    <v-card>
      <v-card-title class="mb-4">
        <v-icon start>mdi-calendar-clock</v-icon>
        Reporte de Próximas Citas
      </v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" md="6">
            <v-text-field
              v-model="searchVet"
              label="Buscar por nombre de Veterinario"
              prepend-inner-icon="mdi-magnify"
              clearable
              @click:clear="loadReport"
              @keyup.enter="loadReport"
              hint="Presiona Enter para buscar"
            />
          </v-col>
          <v-col cols="12" md="2">
            <v-btn color="primary" height="56" block @click="loadReport">
              Consultar
            </v-btn>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="reportData"
          :loading="loading"
          item-value="_id"
          show-expand
        >
          <template #item.veterinario="{ item }">
            <div class="font-weight-bold">{{ item._id.veterinario }}</div>
          </template>

          <template #item.fecha="{ item }">
            <v-chip color="primary" variant="outlined">
              {{ formatDate(item._id.dia) }}
            </v-chip>
          </template>

          <template #item.cantidad="{ item }">
            {{ item.cantidad_citas }} pacientes
          </template>

          <template #expanded-row="{ columns, item }">
            <tr>
              <td :colspan="columns.length" class="bg-grey-lighten-4 pa-4">
                <div class="text-subtitle-2 mb-2">Agenda del día:</div>
                
                <v-table density="compact" class="bg-transparent">
                  <thead>
                    <tr>
                      <th>Hora</th>
                      <th>Paciente</th>
                      <th>Motivo</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="(cita, index) in item.citas" :key="index">
                      <td class="font-weight-medium">{{ cita.hora }}</td>
                      <td>{{ cita.paciente }}</td>
                      <td class="text-grey-darken-1">{{ cita.motivo || '—' }}</td>
                    </tr>
                  </tbody>
                </v-table>
              </td>
            </tr>
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const reportData = ref([]);
const loading = ref(false);
const searchVet = ref('');

// Definimos las columnas de la tabla padre
const headers = [
  { title: 'Veterinario', key: 'veterinario', sortable: true },
  { title: 'Fecha', key: 'fecha', sortable: true },
  { title: 'Carga de Trabajo', key: 'cantidad', align: 'center' },
  { title: '', key: 'data-table-expand' }, // Flechita para expandir
];

async function loadReport() {
  loading.value = true;
  try {
    // Llamada al endpoint de agregación
    const params = {};
    if (searchVet.value) {
      params.veterinarian = searchVet.value;
    }

    const res = await axios.get('/api/appointments/reports/upcoming', { params });
    reportData.value = res.data;
  } catch (error) {
    console.error("Error cargando reporte:", error);
  } finally {
    loading.value = false;
  }
}

// Formatear fecha string YYYY-MM-DD a local
function formatDate(dateStr) {
  if (!dateStr) return '';
  // Truco para evitar problemas de zona horaria al parsear YYYY-MM-DD
  const [year, month, day] = dateStr.split('-');
  return `${day}/${month}/${year}`;
}

onMounted(() => {
  loadReport();
});
</script>