<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Owners
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">
                    Crear dueño
                </v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="owners" :loading="loading">
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
                <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} dueño</v-card-title>

                <v-card-text>
                    <v-text-field v-model="form.name" label="Nombre" required />
                    <v-text-field v-model="form.phone" label="Teléfono" />
                    <v-text-field v-model="form.email" label="Email" />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn text @click="dialog = false">Cancelar</v-btn>

                    <v-btn color="primary" @click="saveOwner">
                        Guardar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Delete dialog -->
        <v-dialog v-model="deleteDialog" max-width="400">
            <v-card>
                <v-card-title>Eliminar dueño</v-card-title>
                <v-card-text>
                    ¿Seguro que deseas eliminar a <strong>{{ form.name }}</strong>?
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>

                    <v-btn text @click="deleteDialog = false">Cancelar</v-btn>

                    <v-btn color="red" @click="deleteOwner">
                        Eliminar
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios';

// -----------------------------
// Estado
// -----------------------------
const owners = ref([]);
const loading = ref(false);

const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

const form = reactive({
    _id: null,
    name: '',
    phone: '',
    email: ''
});

// -----------------------------
// Encabezados de la tabla
// -----------------------------
const headers = [
    { title: 'Nombre', key: 'name' },
    { title: 'Teléfono', key: 'phone' },
    { title: 'Email', key: 'email' },
    { title: 'Acciones', key: 'actions', sortable: false },
];

// -----------------------------
// Cargar datos
// -----------------------------
async function loadOwners() {
    loading.value = true;
    try {
        const res = await axios.get('/api/owners');
        owners.value = res.data;
    } catch (error) {
        console.error('Error loading owners:', error);
    }
    loading.value = false;
}

// onMounted(loadOwners);

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
    form.name = item.name;
    form.phone = item.phone;
    form.email = item.email;
    dialog.value = true;
}

// -----------------------------
// Eliminar
// -----------------------------
function openDeleteDialog(item) {
    form._id = item._id;
    form.name = item.name;
    deleteDialog.value = true;
}

// -----------------------------
// Guardar (crear/editar)
// -----------------------------
async function saveOwner() {
    try {
        if (isEditing.value) {
            await axios.put(`/api/owners/${form._id}`, {
                name: form.name,
                phone: form.phone,
                email: form.email
            });
        } else {
            await axios.post('/api/owners', {
                name: form.name,
                phone: form.phone,
                email: form.email
            });
        }

        dialog.value = false;
        loadOwners();
    } catch (error) {
        console.error("Error saving owner:", error);
    }
}

// -----------------------------
// Confirmar eliminar
// -----------------------------
async function deleteOwner() {
    try {
        await axios.delete(`/api/owners/${form._id}`);
        deleteDialog.value = false;
        loadOwners();
    } catch (error) {
        console.error("Error deleting owner:", error);
    }
}

// -----------------------------
// Reset form
// -----------------------------
function resetForm() {
    form._id = null;
    form.name = '';
    form.phone = '';
    form.email = '';
}
</script>
