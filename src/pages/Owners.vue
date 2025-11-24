<template>
    <div class="pa-4">
        <v-card>
            <v-card-title>
                Dueños
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openCreateDialog">
                    Crear dueño
                </v-btn>
            </v-card-title>

            <v-data-table :headers="headers" :items="owners" :loading="loading">
                <template #item.contacto="{ item }">
                    <div>{{ item.contacto?.telefono || '—' }}</div>
                    <div class="text-caption text-grey">{{ item.contacto?.email }}</div>
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

        <v-dialog v-model="dialog" max-width="500">
            <v-card>
                <v-card-title>{{ isEditing ? 'Editar' : 'Crear' }} dueño</v-card-title>

                <v-card-text>
                    <v-text-field v-model="form.nombre" label="Nombre" required />
                    
                    <v-text-field v-model="form.phone" label="Teléfono" />
                    <v-text-field v-model="form.email" label="Email" />
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="dialog = false">Cancelar</v-btn>
                    <v-btn color="primary" @click="saveOwner">Guardar</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="deleteDialog" max-width="400">
            <v-card>
                <v-card-title>Eliminar dueño</v-card-title>
                <v-card-text>
                    ¿Seguro que deseas eliminar a <strong>{{ form.nombre }}</strong>?
                </v-card-text>

                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn text @click="deleteDialog = false">Cancelar</v-btn>
                    <v-btn color="red" @click="deleteOwner">Eliminar</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios';

// Estado
const owners = ref([]);
const loading = ref(false);
const dialog = ref(false);
const deleteDialog = ref(false);
const isEditing = ref(false);

// Formulario reactivo
const form = reactive({
    _id: null,
    nombre: '', 
    phone: '',
    email: ''
});

// Configuración de tabla
const headers = [
    { title: 'Nombre', key: 'nombre' },
    { title: 'Contacto', key: 'contacto', sortable: false }, // Agrupamos contacto
    { title: 'Acciones', key: 'actions', sortable: false },
];

// Cargar datos
async function loadOwners() {
    loading.value = true;
    try {
        const res = await axios.get('/api/owners');
        owners.value = res.data;
    } catch (error) {
        console.error('Error cargando dueños:', error);
    } finally {
        loading.value = false;
    }
}

onMounted(loadOwners);

// Abrir diálogo Crear
function openCreateDialog() {
    isEditing.value = false;
    resetForm();
    dialog.value = true;
}

// Abrir diálogo Editar
function openEditDialog(item) {
    isEditing.value = true;
    form._id = item._id;
    form.nombre = item.nombre;
    // Mapear datos anidados al formulario plano
    form.phone = item.contacto?.telefono || '';
    form.email = item.contacto?.email || '';
    dialog.value = true;
}

// Abrir diálogo Eliminar
function openDeleteDialog(item) {
    form._id = item._id;
    form.nombre = item.nombre;
    deleteDialog.value = true;
}

// Guardar (Crear o Editar)
async function saveOwner() {
    try {
        // Estructura que espera el Backend (NestJS Schema)
        const payload = {
            nombre: form.nombre,
            contacto: {
                telefono: form.phone,
                email: form.email
            }
        };

        if (isEditing.value) {
            // OJO: Necesitas implementar el método PUT en tu backend para que esto funcione
            await axios.put(`/api/owners/${form._id}`, payload);
        } else {
            await axios.post('/api/owners', payload);
        }

        dialog.value = false;
        loadOwners(); // Recargar tabla
    } catch (error) {
        console.error("Error guardando dueño:", error);
        alert("Error al guardar: " + (error.response?.data?.message || error.message));
    }
}

// Eliminar
async function deleteOwner() {
    try {
        // OJO: Necesitas implementar el método DELETE en tu backend para que esto funcione
        await axios.delete(`/api/owners/${form._id}`);
        deleteDialog.value = false;
        loadOwners();
    } catch (error) {
        console.error("Error eliminando dueño:", error);
        alert("Error al eliminar");
    }
}

// Limpiar formulario
function resetForm() {
    form._id = null;
    form.nombre = '';
    form.phone = '';
    form.email = '';
}
</script>