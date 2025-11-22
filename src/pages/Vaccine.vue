<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Vacunas
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">
                    Nueva vacuna
                </v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="items" :loading="loading">
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

        <!-- Dialog -->
        <v-dialog v-model="dialog" max-width="400">
            <v-card>
                <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} vacuna</v-card-title>

                <v-card-text>
                    <v-text-field v-model="form.nombre" label="Nombre" required />
                    <v-text-field type="number" v-model="form.stock" label="Stock" required />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="dialog = false">Cancelar</v-btn>
                    <v-btn color="primary" @click="save">Guardar</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Delete -->
        <v-dialog v-model="deleteDialog" max-width="350">
            <v-card>
                <v-card-title>Eliminar vacuna</v-card-title>
                <v-card-text>¿Eliminar la vacuna <b>{{ form.nombre }}</b>?</v-card-text>

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

const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);
const loading = ref(false);

const form = reactive({
    _id: null,
    nombre: "",
    stock: 0,
});

const headers = [
    { title: "Nombre", key: "nombre" },
    { title: "Stock", key: "stock" },
    { title: "Acciones", key: "actions", sortable: false },
];

async function loadData() {
    loading.value = true;
    const res = await axios.get("/api/vacunas");
    items.value = res.data;
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
    form.nombre = item.nombre;
    deleteDialog.value = true;
}

async function save() {
    const payload = { nombre: form.nombre, stock: form.stock };

    if (isEditing.value)
        await axios.put(`/api/vacunas/${form._id}`, payload);
    else
        await axios.post("/api/vacunas", payload);

    dialog.value = false;
    loadData();
}

async function remove() {
    await axios.delete(`/api/vacunas/${form._id}`);
    deleteDialog.value = false;
    loadData();
}

function resetForm() {
    form._id = null;
    form.nombre = "";
    form.stock = 0;
}
</script>
