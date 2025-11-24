<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Inventario de Vacunas
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">
                    Nueva vacuna
                </v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="items" :loading="loading">
                <template #item.stock="{ item }">
                    <v-chip :color="getStockColor(item.stock)" size="small">
                        {{ item.stock }} dosis
                    </v-chip>
                </template>

                <template #item.actions="{ item }">
                    <v-btn icon variant="text" size="small" color="primary" @click="openEditDialog(item)">
                        <v-icon>mdi-pencil</v-icon>
                    </v-btn>

                    <v-btn icon variant="text" size="small" color="red" @click="openDeleteDialog(item)">
                        <v-icon>mdi-delete</v-icon>
                    </v-btn>
                </template>
            </v-data-table>
        </v-card>

        <v-dialog v-model="dialog" max-width="400">
            <v-card>
                <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} vacuna</v-card-title>

                <v-card-text>
                    <v-text-field v-model="form.nombre" label="Nombre de la vacuna" required />
                    <v-text-field type="number" v-model.number="form.stock" label="Stock disponible" min="0" required />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="dialog = false">Cancelar</v-btn>
                    <v-btn color="primary" @click="save">Guardar</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteDialog" max-width="350">
            <v-card>
                <v-card-title>Eliminar vacuna</v-card-title>
                <v-card-text>¿Eliminar <b>{{ form.nombre }}</b> del inventario?</v-card-text>

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

// Helper visual para alertar stock bajo
function getStockColor(stock) {
    if (stock === 0) return 'red';
    if (stock < 10) return 'orange';
    return 'green';
}

async function loadData() {
    loading.value = true;
    try {
        // CORRECCIÓN: Ruta en inglés
        const res = await axios.get("/api/vaccines");
        items.value = res.data;
    } catch (error) {
        console.error("Error cargando vacunas:", error);
    } finally {
        loading.value = false;
    }
}

onMounted(loadData);

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
    try {
        const payload = { 
            nombre: form.nombre, 
            stock: Number(form.stock) // Asegurar número
        };

        if (isEditing.value)
            await axios.put(`/api/vaccines/${form._id}`, payload);
        else
            await axios.post("/api/vaccines", payload);

        dialog.value = false;
        loadData();
    } catch (error) {
        console.error("Error guardando:", error);
    }
}

async function remove() {
    try {
        await axios.delete(`/api/vaccines/${form._id}`);
        deleteDialog.value = false;
        loadData();
    } catch (error) {
        console.error("Error eliminando:", error);
        // Aquí podrías validar si la vacuna está en uso antes de borrar
        alert("No se puede eliminar si ya ha sido usada en tratamientos históricos.");
    }
}

function resetForm() {
    form._id = null;
    form.nombre = "";
    form.stock = 0;
}
</script>