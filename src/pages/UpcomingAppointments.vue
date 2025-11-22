<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>
        Próximas citas
      </v-card-title>

      <v-card-text>
        <v-row>
          <v-col cols="12" md="4">
            <v-select
              v-model="selectedVet"
              :items="vets"
              label="Veterinario"
              item-title="name"
              item-value="_id"
            />
          </v-col>

          <v-col cols="12" md="4">
            <v-text-field
              v-model="selectedDate"
              type="date"
              label="Fecha"
            />
          </v-col>

          <v-col cols="12" md="4">
            <v-btn color="primary" block @click="loadAppointments">
              Buscar
            </v-btn>
          </v-col>
        </v-row>

        <v-data-table
          :headers="headers"
          :items="appointments"
          class="mt-4"
        />
      </v-card-text>
    </v-card>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const vets = ref([]);
const appointments = ref([]);

const selectedVet = ref(null);
const selectedDate = ref(null);

const headers = [
  { title: 'Mascota', key: 'pet' },
  { title: 'Fecha', key: 'fecha' },
  { title: 'Motivo', key: 'motivo' },
  { title: 'Estado', key: 'estado' }
];

async function loadVets() {
  const res = await axios.get('/api/vets');
  vets.value = res.data;
}
// loadVets();

async function loadAppointments() {
  const res = await axios.get('/api/reports/upcoming-appointments', {
    params: {
      vet: selectedVet.value,
      date: selectedDate.value
    }
  });

  appointments.value = res.data;
}
</script>
