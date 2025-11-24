<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>
        Agenda de Citas
        <v-spacer></v-spacer>
        <v-btn color="primary" @click="openCreateDialog">Crear cita</v-btn>
      </v-card-title>

      <v-data-table :headers="headers" :items="citas" :loading="loading">
        <template #item.mascota="{ item }">
          {{ item.mascota_id?.nombre || '—' }}
        </template>

        <template #item.estado="{ item }">
          <v-chip :color="getStatusColor(item.estado)" size="small" class="text-capitalize">
            {{ item.estado }}
          </v-chip>
        </template>

        <template #item.fecha="{ item }">
          {{ formatDate(item.fecha) }}
        </template>

        <template #item.actions="{ item }">
          <v-btn 
            v-if="item.estado !== 'cancelada'"
            icon variant="text" size="small" color="primary" 
            @click="openEditDialog(item)"
          >
            <v-icon>mdi-pencil</v-icon>
          </v-btn>

          <v-btn 
            v-if="item.estado !== 'cancelada'"
            icon variant="text" size="small" color="red" 
            @click="openCancelDialog(item)"
            title="Cancelar cita"
          >
            <v-icon>mdi-cancel</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>

    <v-dialog v-model="dialog" max-width="550">
      <v-card>
        <v-card-title>{{ isEditing ? 'Reprogramar' : 'Agendar' }} cita</v-card-title>

        <v-card-text>
          <v-select
            label="Paciente (Mascota)"
            v-model="form.mascota_id"
            :items="pets"
            item-title="nombre"
            item-value="_id"
            required
          />

          <v-text-field type="datetime-local" v-model="form.fecha" label="Fecha y Hora" />

          <v-text-field v-model="form.motivo" label="Motivo" />

          <v-text-field v-model="form.veterinario" label="Veterinario asignado" />

          <v-select
            v-if="isEditing"
            v-model="form.estado"
            label="Estado Actual"
            :items="['pendiente', 'completada', 'cancelada']"
          ></v-select>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="dialog = false">Cerrar</v-btn>
          <v-btn color="primary" @click="saveCita">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="cancelDialog" max-width="400">
      <v-card>
        <v-card-title>Cancelar Cita</v-card-title>
        <v-card-text>
          ¿Estás seguro de cancelar la cita de <strong>{{ selectedItem?.mascota_id?.nombre }}</strong>?
          <br> <small>Esta acción liberará el horario del veterinario.</small>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="cancelDialog = false">Volver</v-btn>
          <v-btn color="red" @click="cancelCitaConfirm">Cancelar Cita</v-btn>
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
const cancelDialog = ref(false);
const isEditing = ref(false);
const selectedItem = ref(null);

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

// --- Carga de Datos ---
async function loadData() {
  loading.value = true;
  try {
    const [citasRes, petsRes] = await Promise.all([
      axios.get("/api/appointments"), 
      axios.get("/api/pets"),
    ]);
    citas.value = citasRes.data;
    pets.value = petsRes.data;
  } catch (error) {
    console.error("Error cargando datos:", error);
  } finally {
    loading.value = false;
  }
}

onMounted(loadData);

// --- Helpers Visuales ---
function getStatusColor(estado) {
  if (estado === 'cancelada') return 'red';
  if (estado === 'completada') return 'green';
  return 'blue';
}

function formatDate(dateString) {
  if (!dateString) return '';
  return new Date(dateString).toLocaleString();
}

// --- Acciones ---

function openCreateDialog() {
  isEditing.value = false;
  resetForm();
  dialog.value = true;
}

function openEditDialog(item) {
  isEditing.value = true;
  form._id = item._id;
  // El backend devuelve el objeto mascota completo, extraemos solo el ID para el select
  form.mascota_id = item.mascota_id?._id || item.mascota_id;
  form.motivo = item.motivo;
  form.veterinario = item.veterinario;
  form.estado = item.estado; // Cargar el estado actual
  
  // Formato para input datetime-local (YYYY-MM-DDTHH:mm)
  if (item.fecha) {
    const date = new Date(item.fecha);
    // Ajuste simple para zona horaria local en el input
    date.setMinutes(date.getMinutes() - date.getTimezoneOffset());
    form.fecha = date.toISOString().slice(0, 16);
  }
  
  dialog.value = true;
}

function openCancelDialog(item) {
  selectedItem.value = item;
  cancelDialog.value = true;
}

// Guardar (Crear o Reprogramar)
async function saveCita() {
  try {
    const payload = {
      mascota_id: form.mascota_id,
      fecha: new Date(form.fecha).toISOString(),
      motivo: form.motivo,
      veterinario: form.veterinario,
      estado: form.estado, // Enviamos el estado por si cambió
    };

    if (isEditing.value) {
      // PUT /api/appointments/:id
      await axios.put(`/api/appointments/${form._id}`, payload);
    } else {
      // POST /api/appointments
      await axios.post("/api/appointments", payload);
    }

    dialog.value = false;
    loadData();
  } catch (error) {
    console.error("Error guardando cita:", error);
    const msg = error.response?.data?.message || error.message;
    alert("Error: " + msg);
  }
}

// Cancelar Cita (Lógica especial)
async function cancelCitaConfirm() {
  if (!selectedItem.value) return;
  
  try {
    // Usamos PATCH .../cancel como definimos en el backend
    await axios.patch(`/api/appointments/${selectedItem.value._id}/cancel`);
    cancelDialog.value = false;
    loadData();
  } catch (error) {
    console.error("Error cancelando cita:", error);
    alert("No se pudo cancelar");
  }
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