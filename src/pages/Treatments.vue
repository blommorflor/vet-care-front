<template>
  <div class="pa-4">
    <v-card>
      <v-card-title>
        Tratamientos
        <v-spacer></v-spacer>
        <v-btn color="primary" @click="openCreateDialog">Nuevo Tratamiento</v-btn>
      </v-card-title>

      <v-data-table :headers="headers" :items="items" :loading="loading">
        <template #item.mascota="{ item }">
          {{ item.mascota_id?.nombre || '—' }}
        </template>

        <template #item.insumos="{ item }">
          <div v-if="item.vacunas_usadas && item.vacunas_usadas.length > 0">
            <v-chip 
              v-for="(v, index) in item.vacunas_usadas" 
              :key="index"
              size="x-small" 
              color="blue-lighten-4" 
              class="mr-1 mb-1"
            >
              {{ v.vacuna_id?.nombre }} ({{ v.cantidad }})
            </v-chip>
          </div>
          <span v-else class="text-grey text-caption">Sin insumos</span>
        </template>

        <template #item.fecha="{ item }">
          {{ formatDate(item.fecha) }}
        </template>

        <template #item.costo="{ item }">
          ${{ item.costo }}
        </template>

        <template #item.actions="{ item }">
           <v-btn icon variant="text" size="small" color="red" @click="openDeleteDialog(item)">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </template>
      </v-data-table>
    </v-card>

    <v-dialog v-model="dialog" max-width="700">
      <v-card>
        <v-card-title>Registrar Tratamiento</v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" md="6">
              <v-select
                v-model="form.mascota_id"
                label="Paciente"
                :items="pets"
                item-title="nombre"
                item-value="_id"
              />
            </v-col>
            <v-col cols="12" md="6">
              <v-text-field type="datetime-local" v-model="form.fecha" label="Fecha" />
            </v-col>
          </v-row>

          <v-row>
             <v-col cols="12" md="8">
                <v-text-field v-model="form.tipo" label="Tipo" />
             </v-col>
             <v-col cols="12" md="4">
                <v-text-field type="number" prefix="$" label="Costo Total" v-model.number="form.costo" />
             </v-col>
          </v-row>

          <v-textarea v-model="form.notas" label="Notas clínicas" rows="2" />

          <v-divider class="my-3"></v-divider>
          
          <div class="d-flex justify-space-between align-center mb-2">
            <div class="text-subtitle-1 font-weight-bold">Uso de Vacunas/Insumos</div>
            <v-btn size="small" color="secondary" variant="tonal" prepend-icon="mdi-plus" @click="addInsumoRow">
              Agregar
            </v-btn>
          </div>

          <v-table density="compact" class="border rounded">
            <thead>
              <tr>
                <th>Vacuna</th>
                <th style="width: 100px">Dosis</th>
                <th style="width: 50px"></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, index) in form.insumos" :key="index">
                <td>
                  <v-select
                    v-model="row.vacuna_id"
                    :items="vacunas"
                    item-title="nombre"
                    item-value="_id"
                    variant="underlined"
                    density="compact"
                    hide-details
                    placeholder="Seleccionar..."
                  ></v-select>
                </td>
                <td>
                  <v-text-field
                    type="number"
                    v-model.number="row.cantidad"
                    min="1"
                    variant="underlined"
                    density="compact"
                    hide-details
                  ></v-text-field>
                </td>
                <td>
                  <v-btn icon size="x-small" variant="text" color="red" @click="removeInsumoRow(index)">
                    <v-icon>mdi-close</v-icon>
                  </v-btn>
                </td>
              </tr>
              <tr v-if="form.insumos.length === 0">
                <td colspan="3" class="text-center text-grey text-caption py-2">
                  No se han agregado vacunas a este tratamiento.
                </td>
              </tr>
            </tbody>
          </v-table>

        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="dialog = false">Cancelar</v-btn>
          <v-btn color="primary" @click="save" :loading="saving">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="deleteDialog" max-width="400">
       <v-card>
        <v-card-title>Eliminar registro</v-card-title>
        <v-card-text>¿Borrar este tratamiento?</v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn text @click="deleteDialog = false">Cancelar</v-btn>
          <v-btn color="red" @click="remove">Eliminar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from "vue";
import axios from "axios";

const items = ref([]);
const pets = ref([]);
const vacunas = ref([]);

const loading = ref(false);
const saving = ref(false);
const dialog = ref(false);
const deleteDialog = ref(false);

const form = reactive({
  _id: null,
  mascota_id: "",
  fecha: "",
  tipo: "",
  notas: "",
  costo: 0,
  insumos: [] // Array de { vacuna_id, cantidad }
});

const headers = [
  { title: "Mascota", key: "mascota" },
  { title: "Fecha", key: "fecha" },
  { title: "Tipo", key: "tipo" },
  { title: "Insumos Utilizados", key: "insumos" }, // Nueva columna
  { title: "Costo", key: "costo" },
  { title: "Acciones", key: "actions", sortable: false },
];

function formatDate(dateStr) {
  if(!dateStr) return '';
  return new Date(dateStr).toLocaleString();
}

async function loadData() {
    loading.value = true;
    try {
        const [t1, t2, t3] = await Promise.all([
            axios.get("/api/treatments"),
            axios.get("/api/pets"),
            axios.get("/api/vaccines"),
        ]);

        items.value = t1.data;
        pets.value = t2.data;
        vacunas.value = t3.data;
    } catch (error) {
        console.error("Error cargando datos:", error);
    } finally {
        loading.value = false;
    }
}

onMounted(loadData);

// --- Lógica de Insumos Múltiples ---
function addInsumoRow() {
  form.insumos.push({ vacuna_id: null, cantidad: 1 });
}

function removeInsumoRow(index) {
  form.insumos.splice(index, 1);
}

// --- Acciones CRUD ---

function openCreateDialog() {
  resetForm();
  const now = new Date();
  now.setMinutes(now.getMinutes() - now.getTimezoneOffset());
  form.fecha = now.toISOString().slice(0, 16);
  dialog.value = true;
}

function openDeleteDialog(item) {
  form._id = item._id;
  deleteDialog.value = true;
}

async function save() {
  saving.value = true;
  try {
    // Filtramos filas vacías (que no tengan vacuna seleccionada)
    const vacunasValidas = form.insumos.filter(row => row.vacuna_id && row.cantidad > 0);

    const payload = {
      mascota_id: form.mascota_id,
      fecha: new Date(form.fecha).toISOString(),
      tipo: form.tipo,
      notas: form.notas,
      costo: form.costo,
      vacunas_usadas: vacunasValidas // Enviamos el array al backend
    };

    await axios.post("/api/treatments", payload);
    
    dialog.value = false;
    loadData();
  } catch (error) {
    console.error("Error guardando:", error);
    const msg = error.response?.data?.message || error.message;
    alert("Error: " + msg);
  } finally {
    saving.value = false;
  }
}

async function remove() {
  try {
    await axios.delete(`/api/treatments/${form._id}`);
    deleteDialog.value = false;
    loadData();
  } catch (error) { console.error(error); }
}

function resetForm() {
  form._id = null;
  form.mascota_id = "";
  form.fecha = "";
  form.tipo = "";
  form.notas = "";
  form.costo = 0;
  form.insumos = [];
}
</script>