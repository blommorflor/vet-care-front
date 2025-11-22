<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>
        Mascotas con esquema de vacunas incompleto
      </v-card-title>

      <v-data-table
        :headers="headers"
        :items="pets"
        class="ma-4"
      >
        <template #item.pendientes="{ item }">
          <v-chip
            v-for="vac in item.pendientes"
            :key="vac"
            color="red"
            class="ma-1"
          >
            {{ vac }}
          </v-chip>
        </template>
      </v-data-table>
    </v-card>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

const pets = ref([]);

const headers = [
  { title: 'Mascota', key: 'nombre' },
  { title: 'Dueño', key: 'dueno' },
  { title: 'Vacunas pendientes', key: 'pendientes' }
];

async function loadPets() {
  const res = await axios.get('/api/reports/incomplete-vaccines');
  pets.value = res.data;
}

// onMounted(loadPets);
</script>
