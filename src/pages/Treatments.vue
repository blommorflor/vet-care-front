<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Tratamientos
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">Crear tratamiento</v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="items" :loading="loading">
                <template #item.mascota="{ item }">
                    {{ getPetName(item.mascota_id) }}
                </template>

                <template #item.vacuna="{ item }">
                    {{ getVaccineName(item.vacuna_usada?.vacuna_id) }}
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

        <!-- Dialog crear/editar -->
        <v-dialog v-model="dialog" max-width="550">
            <v-card>
                <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} tratamiento</v-card-title>

                <v-card-text>
                    <v-select v-model="form.mascota_id" label="Mascota" :items="pets" item-title="nombre"
                        item-value="_id" />

                    <v-text-field type="date" v-model="form.fecha" label="Fecha" />

                    <v-text-field v-model="form.tipo" label="Tipo de tratamiento" />

                    <v-textarea v-model="form.notas" label="Notas (opcional)" />

                    <v-text-field type="number" label="Costo" v-model="form.costo" />

                    <v-divider class="my-3" />

                    <v-select v-model="form.vacuna_usada.vacuna_id" label="Vacuna usada" :items="vacunas"
                        item-title="nombre" item-value="_id" />

                    <v-text-field type="number" v-model="form.vacuna_usada.cantidad" label="Cantidad usada" />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="dialog = false">Cancelar</v-btn>
                    <v-btn color="primary" @click="save">Guardar</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Eliminar -->
        <v-dialog v-model="deleteDialog" max-width="400">
            <v-card>
                <v-card-title>Eliminar tratamiento</v-card-title>
                <v-card-text>¿Seguro que deseas eliminar este tratamiento?</v-card-text>

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
const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

const form = reactive({
    _id: null,
    mascota_id: "",
    fecha: "",
    tipo: "",
    notas: "",
    costo: "",
    vacuna_usada: {
        vacuna_id: "",
        cantidad: "",
    },
});

const headers = [
    { title: "Mascota", key: "mascota" },
    { title: "Fecha", key: "fecha" },
    { title: "Tipo", key: "tipo" },
    { title: "Costo", key: "costo" },
    { title: "Vacuna usada", key: "vacuna" },
    { title: "Acciones", key: "actions", sortable: false },
];

function getPetName(id) {
    return pets.value.find(p => p._id === id)?.nombre || "—";
}

function getVaccineName(id) {
    return vacunas.value.find(v => v._id === id)?.nombre || "—";
}

async function loadData() {
    loading.value = true;

    const [t1, t2, t3] = await Promise.all([
        axios.get("/api/tratamientos"),
        axios.get("/api/pets"),
        axios.get("/api/vacunas"),
    ]);

    items.value = t1.data;
    pets.value = t2.data;
    vacunas.value = t3.data;

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

async function save() {
    const payload = JSON.parse(JSON.stringify(form)); // copiar limpio

    if (isEditing.value)
        await axios.put(`/api/tratamientos/${form._id}`, payload);
    else
        await axios.post("/api/tratamientos", payload);

    dialog.value = false;
    loadData();
}

async function remove() {
    await axios.delete(`/api/tratamientos/${form._id}`);
    deleteDialog.value = false;
    loadData();
}

function resetForm() {
    Object.assign(form, {
        _id: null,
        mascota_id: "",
        fecha: "",
        tipo: "",
        notas: "",
        costo: "",
        vacuna_usada: { vacuna_id: "", cantidad: "" },
    });
}
</script>
