<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Pets
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">
                    Crear mascota
                </v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="pets" :loading="loading">
                <template #item.owner="{ item }">
                    {{ getOwnerName(item.dueno_id) }}
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

        <!-- Create/Edit dialog -->
        <v-dialog v-model="dialog" max-width="500">
            <v-card>
                <v-card-title>
                    {{ isEditing ? 'Editar mascota' : 'Crear mascota' }}
                </v-card-title>

                <v-card-text>
                    <v-text-field v-model="form.nombre" label="Nombre" required />

                    <v-text-field v-model="form.especie" label="Especie" required />

                    <v-text-field v-model="form.raza" label="Raza (opcional)" />

                    <v-select v-model="form.dueno_id" :items="owners" item-title="name" item-value="_id" label="Dueño"
                        required />

                    <v-text-field v-model="form.fecha_nacimiento" type="date" label="Fecha de nacimiento" />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn text @click="dialog = false">Cancelar</v-btn>

                    <v-btn color="primary" @click="savePet">
                        Guardar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Delete dialog -->
        <v-dialog v-model="deleteDialog" max-width="400">
            <v-card>
                <v-card-title>Eliminar mascota</v-card-title>
                <v-card-text>
                    ¿Seguro que deseas eliminar a <strong>{{ form.nombre }}</strong>?
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn text @click="deleteDialog = false">Cancelar</v-btn>

                    <v-btn color="red" @click="deletePet">
                        Eliminar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted } from "vue";
import axios from "axios";

// -----------------------------
// Estado
// -----------------------------
const pets = ref([]);
const owners = ref([]);
const loading = ref(false);

const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

const form = reactive({
    _id: null,
    nombre: "",
    especie: "",
    raza: "",
    dueno_id: "",
    fecha_nacimiento: "",
});

// -----------------------------
// Encabezados de la tabla
// -----------------------------
const headers = [
    { title: "Nombre", key: "nombre" },
    { title: "Especie", key: "especie" },
    { title: "Raza", key: "raza" },
    { title: "Dueño", key: "owner" },
    { title: "Fecha Nacimiento", key: "fecha_nacimiento" },
    { title: "Acciones", key: "actions", sortable: false },
];

// -----------------------------
// Helpers
// -----------------------------
function getOwnerName(ownerId) {
    const owner = owners.value.find((o) => o._id === ownerId);
    return owner ? owner.name : "—";
}

// -----------------------------
// Cargar datos
// -----------------------------
async function loadPets() {
    loading.value = true;
    try {
        const res = await axios.get("/api/pets");
        pets.value = res.data;
    } catch (error) {
        console.error("Error loading pets:", error);
    }
    loading.value = false;
}

async function loadOwners() {
    try {
        const res = await axios.get("/api/owners");
        owners.value = res.data;
    } catch (error) {
        console.error("Error loading owners:", error);
    }
}

onMounted(() => {
    // loadPets();
    // loadOwners();
});

// -----------------------------
// Crear
// -----------------------------
function openCreateDialog() {
    isEditing.value = false;
    resetForm();
    dialog.value = true;
}

// -----------------------------
// Editar
// -----------------------------
function openEditDialog(item) {
    isEditing.value = true;

    form._id = item._id;
    form.nombre = item.nombre;
    form.especie = item.especie;
    form.raza = item.raza;
    form.dueno_id = item.dueno_id;
    form.fecha_nacimiento = item.fecha_nacimiento;

    dialog.value = true;
}

// -----------------------------
// Eliminar
// -----------------------------
function openDeleteDialog(item) {
    form._id = item._id;
    form.nombre = item.nombre;
    deleteDialog.value = true;
}

// -----------------------------
// Guardar (crear/editar)
// -----------------------------
async function savePet() {
    try {
        const payload = {
            nombre: form.nombre,
            especie: form.especie,
            raza: form.raza,
            dueno_id: form.dueno_id,
            fecha_nacimiento: form.fecha_nacimiento,
        };

        if (isEditing.value) {
            await axios.put(`/api/pets/${form._id}`, payload);
        } else {
            await axios.post("/api/pets", payload);
        }

        dialog.value = false;
        loadPets();
    } catch (error) {
        console.error("Error saving pet:", error);
    }
}

// -----------------------------
// Confirmar eliminar
// -----------------------------
async function deletePet() {
    try {
        await axios.delete(`/api/pets/${form._id}`);
        deleteDialog.value = false;
        loadPets();
    } catch (error) {
        console.error("Error deleting pet:", error);
    }
}

// -----------------------------
// Reiniciar form
// -----------------------------
function resetForm() {
    form._id = null;
    form.nombre = "";
    form.especie = "";
    form.raza = "";
    form.dueno_id = "";
    form.fecha_nacimiento = "";
}
</script>
