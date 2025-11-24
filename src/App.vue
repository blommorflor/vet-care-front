<template>
  <v-app>
    <v-main>
      <v-app-bar :elevation="3" color="secondary">
        <template v-slot:prepend>
          <v-app-bar-nav-icon @click="handleOpenDrawe()"></v-app-bar-nav-icon>
        </template>

        <v-app-bar-title>VetCare</v-app-bar-title>

        <template v-slot:append>
          <v-btn icon="mdi-heart"></v-btn>

          <v-btn icon="mdi-magnify"></v-btn>

          <v-btn icon="mdi-dots-vertical"></v-btn>
        </template>
      </v-app-bar>
      <v-navigation-drawer v-model="open">
        <v-list-item link title="Dueños" to="/owners" prepend-icon="mdi-account-group"></v-list-item>

        <v-list-item link title="Mascotas" to="/pets" prepend-icon="mdi-dog"></v-list-item>

        <v-list-item link title="Citas" to="/appoiments" prepend-icon="mdi-calendar-check"></v-list-item>

        <v-list-item link title="Tratamientos" to="/treatments" prepend-icon="mdi-medical-bag"></v-list-item>

        <v-list-item link title="Vacunas" to="/vaccine" prepend-icon="mdi-needle"></v-list-item>

        <v-list-item link title="Citas por veterinario" to="/upcomingAppointments" prepend-icon="mdi-calendar-clock"></v-list-item>

        <v-list-item link title="Gasto por mascota" to="/monthlyCostByPet" prepend-icon="mdi-cash-multiple"></v-list-item>

        <v-list-item link title="Mascotas Vacunas" to="/incompleteVaccines" prepend-icon="mdi-alert-circle-outline"></v-list-item>
      </v-navigation-drawer>
      <div style="padding: 15px; background-color: ghostwhite; height: 100%; width: 100%;">
        <v-card style="padding: 15px;">
          <router-view />
        </v-card>
      </div>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'

const open = ref(true)

function handleOpenDrawe() {
  open.value = !open.value
}
</script>
