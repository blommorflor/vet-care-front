<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>Gasto Mensual por Mascota</v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" md="4">
            <v-select
              v-model="selectedPet"
              :items="pets"
              label="Mascota"
              item-title="nombre"
              item-value="_id"
            />
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field
              v-model="selectedMonth"
              type="month"
              label="Mes"
            />
          </v-col>

          <v-col cols="12" md="4">
            <v-btn color="primary" block @click="loadCosts">
              Consultar
            </v-btn>
          </v-col>
        </v-row>

        <v-alert
          v-if="totalCost !== null"
          type="info"
          class="mt-4"
        >
          Gasto total del mes: <strong>\${{ totalCost }}</strong>
        </v-alert>

        <v-data-table
          :headers="headers"
          :items="treatments"
          class="mt-4"
        />
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const pets = ref([]);
const treatments = ref([]);
const totalCost = ref(null);

const selectedPet = ref(null);
const selectedMonth = ref(null);

const headers = [
  { title: 'Fecha', key: 'fecha' },
  { title: 'Tipo', key: 'tipo' },
  { title: 'Costo', key: 'costo' },
];

async function loadPets() {
  const res = await axios.get('/api/pets');
  pets.value = res.data;
}
// loadPets();

async function loadCosts() {
  const res = await axios.get('/api/reports/monthly-cost', {
    params: {
      pet: selectedPet.value,
      month: selectedMonth.value
    }
  });

  treatments.value = res.data.items;
  totalCost.value = res.data.total;
}
</script>
