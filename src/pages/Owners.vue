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

const owners = ref([]);
const loading = ref(false);
const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

// CORRECCIÓN 1: Usar 'nombre' en lugar de 'name'
const form = reactive({
    _id: null,
    nombre: '', 
    phone: '',
    email: ''
});

// CORRECCIÓN 2: La key debe coincidir con la base de datos ('nombre')
const headers = [
    { title: 'Nombre', key: 'nombre' }, 
    { title: 'Teléfono', key: 'contacto.telefono' }, // Acceso al objeto anidado
    { title: 'Email', key: 'contacto.email' },       // Acceso al objeto anidado
    { title: 'Acciones', key: 'actions', sortable: false },
];

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

// Descomentar para que cargue al iniciar
onMounted(loadOwners);

function openCreateDialog() {
    isEditing.value = false;
    resetForm();
    dialog.value = true;
}

function openEditDialog(item) {
    isEditing.value = true;
    form._id = item._id;
    form.nombre = item.nombre; // Corregido
    // El backend devuelve contacto: { telefono: '...', email: '...' }
    form.phone = item.contacto?.telefono || ''; 
    form.email = item.contacto?.email || '';
    dialog.value = true;
}

function openDeleteDialog(item) {
    form._id = item._id;
    form.nombre = item.nombre; // Corregido
    deleteDialog.value = true;
}

async function saveOwner() {
    try {
        // Adaptar el payload a la estructura que espera el Backend
        // duenos(_id, nombre, contacto: { telefono, email })
        const payload = {
            nombre: form.nombre,
            contacto: {
                telefono: form.phone,
                email: form.email
            }
        };

        if (isEditing.value) {
            await axios.put(`/api/owners/${form._id}`, payload); // Asegúrate de tener PUT en tu controller si usas editar
        } else {
            await axios.post('/api/owners', payload);
        }

        dialog.value = false;
        loadOwners();
    } catch (error) {
        console.error("Error saving owner:", error);
    }
}

async function deleteOwner() {
    try {
        await axios.delete(`/api/owners/${form._id}`); // Asegúrate de tener DELETE en tu controller
        deleteDialog.value = false;
        loadOwners();
    } catch (error) {
        console.error("Error deleting owner:", error);
    }
}

function resetForm() {
    form._id = null;
    form.nombre = '';
    form.phone = '';
    form.email = '';
}
</script>
