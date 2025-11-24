<template>
  <div class="pa-4">
    <v-card>
      <v-card-title class="mb-4">
        <v-icon start color="green">mdi-cash-multiple</v-icon>
        Reporte de Gastos Mensuales
      </v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" md="8">
            <v-text-field
              v-model="searchPet"
              label="Filtrar por nombre de Mascota"
              prepend-inner-icon="mdi-magnify"
              clearable
              @click:clear="loadReport"
              @keyup.enter="loadReport"
              hint="Presiona Enter para buscar"
            />
          </v-col>
          <v-col cols="12" md="4">
             <v-card color="green-lighten-5" variant="flat" class="pa-2 text-center">
                <div class="text-caption text-uppercase font-weight-bold">Total Reportado</div>
                <div class="text-h5 text-green-darken-2 font-weight-black">
                  {{ formatCurrency(grandTotal) }}
                </div>
             </v-card>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="reportData"
          :loading="loading"
          class="elevation-1 mt-4"
        >
          <template #item.mascota="{ item }">
            <div class="d-flex align-center">
              <v-avatar color="grey-lighten-3" size="32" class="mr-2">
                <v-icon size="small">mdi-paw</v-icon>
              </v-avatar>
              <span class="font-weight-bold">{{ item.mascota }}</span>
            </div>
          </template>

          <template #item.mes="{ item }">
            <v-chip size="small" variant="outlined">
              {{ formatMonth(item.mes) }}
            </v-chip>
          </template>

          <template #item.procedimientos="{ item }">
            {{ item.procedimientos }} atenciones
          </template>

          <template #item.total="{ item }">
            <strong class="text-green-darken-2">{{ formatCurrency(item.total) }}</strong>
          </template>
        </v-data-table>
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

const reportData = ref([]);
const loading = ref(false);
const searchPet = ref('');

const headers = [
  { title: 'Mascota', key: 'mascota', sortable: true },
  { title: 'Mes', key: 'mes', sortable: true }, // El backend manda formato YYYY-MM
  { title: 'Procedimientos', key: 'procedimientos', align: 'center' },
  { title: 'Gasto Total', key: 'total', align: 'end' },
];

// Calculamos el total de todo lo que se ve en pantalla
const grandTotal = computed(() => {
  return reportData.value.reduce((acc, item) => acc + (item.total || 0), 0);
});

async function loadReport() {
  loading.value = true;
  try {
    const params = {};
    if (searchPet.value) {
      params.petName = searchPet.value; // El backend espera 'petName'
    }

    // CORRECCIÓN: Endpoint en inglés
    const res = await axios.get('/api/treatments/reports/expenses', { params });
    reportData.value = res.data;
  } catch (error) {
    console.error("Error cargando reporte de gastos:", error);
  } finally {
    loading.value = false;
  }
}

// Helpers de formato
function formatCurrency(value) {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(value);
}

function formatMonth(yyyyMm) {
  if (!yyyyMm) return '';
  // Convierte "2023-11" a "Noviembre 2023" (Depende del locale del navegador)
  const [year, month] = yyyyMm.split('-');
  const date = new Date(year, month - 1); 
  return date.toLocaleDateString(undefined, { month: 'long', year: 'numeric' });
}

onMounted(() => {
  loadReport();
});
</script>