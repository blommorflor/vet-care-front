<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>
        Citas
        <v-spacer></v-spacer>
        <v-btn color="primary" @click="openCreateDialog">Crear cita</v-btn>
      </v-card-title>

      <v-data-table :headers="headers" :items="citas" :loading="loading">
        <template #item.mascota="{ item }">
          {{ getPetName(item.mascota_id) }}
        </template>

        <template #item.actions="{ item }">
          <v-btn icon @click="openEditDialog(item)">
            <v-icon>mdi-pencil</v-icon>
          </v-btn>

          <v-btn icon color="red" @click="openDeleteDialog(item)">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>

    <!-- Create/Edit -->
    <v-dialog v-model="dialog" max-width="550">
      <v-card>
        <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} cita</v-card-title>

        <v-card-text>
          <v-select
            label="Mascota"
            v-model="form.mascota_id"
            :items="pets"
            item-title="nombre"
            item-value="_id"
            required
          />

          <v-text-field type="date" v-model="form.fecha" label="Fecha" />

          <v-text-field v-model="form.motivo" label="Motivo (opcional)" />

          <v-text-field v-model="form.veterinario" label="Veterinario" />

          <v-select
            v-model="form.estado"
            label="Estado"
            :items="['pendiente', 'completado', 'cancelado']"
          />
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="saveCita">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Delete -->
    <v-dialog v-model="deleteDialog" max-width="400">
      <v-card>
        <v-card-title>Eliminar cita</v-card-title>
        <v-card-text>
          ¿Seguro que deseas eliminar esta cita?
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="deleteDialog = false">Cancelar</v-btn>
          <v-btn color="red" @click="deleteCita">Eliminar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from "vue";
import axios from "axios";

const citas = ref([]);
const pets = ref([]);

const loading = ref(false);
const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

const form = reactive({
  _id: null,
  mascota_id: "",
  fecha: "",
  motivo: "",
  veterinario: "",
  estado: "pendiente",
});

const headers = [
  { title: "Mascota", key: "mascota" },
  { title: "Fecha", key: "fecha" },
  { title: "Motivo", key: "motivo" },
  { title: "Veterinario", key: "veterinario" },
  { title: "Estado", key: "estado" },
  { title: "Acciones", key: "actions", sortable: false },
];

function getPetName(id) {
  const pet = pets.value.find((p) => p._id === id);
  return pet ? pet.nombre : "—";
}

async function loadData() {
  loading.value = true;
  const [c1, c2] = await Promise.all([
    axios.get("/api/citas"),
    axios.get("/api/pets"),
  ]);
  citas.value = c1.data;
  pets.value = c2.data;
  loading.value = false;
}

// onMounted(loadData);

function openCreateDialog() {
  isEditing.value = false;
  resetForm();
  dialog.value = true;
}

function openEditDialog(item) {
  isEditing.value = true;
  Object.assign(form, item);
  dialog.value = true;
}

function openDeleteDialog(item) {
  form._id = item._id;
  deleteDialog.value = true;
}

async function saveCita() {
  const payload = { ...form };

  if (isEditing.value)
    await axios.put(`/api/citas/${form._id}`, payload);
  else
    await axios.post("/api/citas", payload);

  dialog.value = false;
  loadData();
}

async function deleteCita() {
  await axios.delete(`/api/citas/${form._id}`);
  deleteDialog.value = false;
  loadData();
}

function resetForm() {
  form._id = null;
  form.mascota_id = "";
  form.fecha = "";
  form.motivo = "";
  form.veterinario = "";
  form.estado = "pendiente";
}
</script>
