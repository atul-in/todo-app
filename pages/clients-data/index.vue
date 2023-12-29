<template>
    <div class="main-container">

        <v-dialog v-model="isInsertDialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="text-h5">Insert Clients Data</span>
                </v-card-title>
                <v-card-text>
                    <v-file-input v-model="file" label="Select file" accept=".csv, .xlsx"></v-file-input>
                    <v-btn @click="uploadFile" :disabled="file === null" :loading="loading" color="blue"
                        type="submit">Upload File</v-btn>
                    <v-btn @click="isInsertDialog = false">Close</v-btn>
                </v-card-text>
            </v-card>
        </v-dialog>

        <!-- Add Client Data Dialog -->
        <v-dialog v-model="addClientDialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="text-h5">Add New Client</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="6">
                                <v-text-field v-model="clientData.firstName" label="First Name" required></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field v-model="clientData.lastName" label="Last Name" required></v-text-field>
                            </v-col>
                            <v-col cols="12">
                                <v-text-field v-model="clientData.email" label="Email" required></v-text-field>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="addClientDialog = false">
                        Close
                    </v-btn>
                    <v-btn color="blue darken-1" :loading="loading" text @click="saveClientData">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- Edit Client Data Dialog -->
        <v-dialog v-model="editClientDialog" persistent max-width="600px">
            <v-card>
                <v-card-title>
                    <span class="text-h5">Edit Client</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col cols="6">
                                <v-text-field v-model="clientData.firstName" label="First Name" required></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field v-model="clientData.lastName" label="Last Name" required></v-text-field>
                            </v-col>
                            <v-col cols="12">
                                <v-text-field v-model="clientData.email" label="Email" required></v-text-field>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="blue darken-1" text @click="editClientDialog = false">
                        Close
                    </v-btn>
                    <v-btn color="blue darken-1" :loading="loading" text @click="updateClientData">
                        Save
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <!-- todo Table -->


        <div class="d-flex align-center absolute justify-center">
            <v-container>
                <v-data-table :loading="loading" :headers="headers" :items="clients" class="elevation-1"
                    :footer-props="{ 'items-per-currentPage-options': [5, 10, 50, 100, -1] }" @update:currentPage="handleCurrentPageUpdate"
                    @update:items-per-currentPage="handleItemsPerPageUpdate" :server-items-length="totalClients" :sort-by.sync="sortBy"
                    :must-sort="true" :sort-desc.sync="sortDesc" @update:sort-desc="fetchClientsData">
                    <template v-slot:top>
                        <v-toolbar flat>
                            <v-toolbar-title>My Clients List</v-toolbar-title>
                            <v-divider class="mx-4" inset vertical></v-divider>
                            <v-spacer></v-spacer>
                            <v-btn class="m-5 mr-5 blue" @click="openInsertDataDialog">+
                                Insert Clients Data
                            </v-btn>
                            <v-btn class="m-5 mr-5 green" @click="downloadClientsData">
                                Download Clients Data
                            </v-btn>
                            <v-btn class="m-5 mr-5 blue--text" @click="openClientDialog">+ Add Client</v-btn>
                        </v-toolbar>
                    </template>
                    <!-- <template #item.index="{ item, index }">{{ calculateSerialNumber(index) }}</template>
                    <template #item.created_at="{ item, created_at }">{{ $dayjs(created_at).format('DD/MM/YYYY')
                    }}</template>
                    <template #item.completed="{ item }"><v-checkbox v-model="item.completed" :disabled="disabled"
                            @change="updatedStatus(item)"></v-checkbox></template>
                    <template v-slot:item.action_buttons="{ item }">

                        <div class="d-flex">
                            <v-btn v-if="userRole != 'admin'" :loading="item.loading" elevation="0" icon color="green"
                                @click="editTodoData(item)">
                                <v-icon dark>mdi-pencil</v-icon>
                            </v-btn>

                            <v-btn v-if="item.id != clientIdToDelete" :loading="item.loading" elevation="0" icon color="red"
                                @click="handleConfirmDialog(item.id)">
                                <v-icon dark>mdi-delete</v-icon>
                            </v-btn>
                            <v-progress-circular v-else indeterminate color="primary"></v-progress-circular>
                        </div>
                    </template> -->
                </v-data-table>
            </v-container>
        </div>

        <v-snackbar v-model="snackbar">
            {{ snackbarText }}

            <template v-slot:action="{ attrs }">
                <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
                    Close
                </v-btn>
            </template>
        </v-snackbar>
    </div>
</template>
  
<script>

export default {
    name: 'ClientsData',

    data() {
        return {

            file: null,
            isInsertDialog: false,
            disabled: this.$auth.user.role[0] == 'admin' ? true : false,
            
            snackbar: false,
            snackbarText: '',
            
            //Paging and Sorting
            sortBy: "created_at",
            sortDesc: true,
            currentPage: 1,
            itemsPerPage: 10,
            totalClients: 0,
            
            loading: false,
            
            // Table Data
            headers: [
                { text: "S.no.", value: "index" },
                { text: "First Name", value: "firstName", sortable: true },
                { text: "Last Name", value: "lastName", sortable: true },
                { text: "Email", value: "email", sortable: true },
                { text: "Created At", value: "created_at", sortable: true },
                { text: "Action", value: "action_buttons", sortable: false },
            ],
            
            progressbar: false,
            userRole: this.$auth.user.role[0],
            
            //Add Data            
            addClientDialog: false,
            clientData: {
                firstName: '',
                lastName: '',
                email: '',
                user_id: this.$auth.user.data.id,
            },

            clients: [],
            
            //Edit Data
            editClientDialog: false,
            clientIdToEdit: 0,
            todoEditTitle: '',
            todoEditDescription: '',

            // Delete Data
            clientIdToDelete: null,
        }
    },
    
    mounted() {
        this.snackbar = false
        this.fetchClientsData()
    },

    methods: {
        calculateSerialNumber(index) {
            return (this.currentPage - 1) * this.itemsPerPage + index + 1;
        },

        async uploadFile() {
            this.loading = true
            if (!this.file) {
                console.log("Please select a file")
                return
            }
            const formData = new FormData()
            formData.append('file', this.file)
            try {
                const response = await this.$axios.post('/api/import-tasks', formData)
                // console.log(response)
                this.snackbarText = response.data.message
                this.snackbar = true
                this.loading = false
                this.isInsertDialog = false
            } catch (error) {
                console.log(error)
                this.snackbarText = error.message
                this.snackbar = true
                this.loading = false
            } finally {
                this.file = null
                this.fetchClientsData()
            }
        },

        openInsertDataDialog() {
            this.isInsertDialog = true;
        },

        async downloadClientsData() {
            // try {
            //     const response = await this.$axios.get('/api/download-tasks', { responseType: 'blob' })
            //     console.log('Fetched', response)
            //     const blob = new Blob([response.data], {
            //         type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
            //     })
            //     console.log('blob', blob)

            //     const link = document.createElement('a');
            //     link.href = URL.createObjectURL(blob);
            //     link.download = 'tasks.xlsx';
            //     document.body.appendChild(link);
            //     link.click();
            //     document.body.removeChild(link);

            // } catch (error) {
            //     console.log(error)
            // }
        },

        handleConfirmDialog(todoId) {
            this.$confirm({
                message: 'Are you sure?',
                button: {
                    no: 'No',
                    yes: 'Yes'
                },
                /**
                 * Callback Function
                 * @param {Boolean} confirm
                 */
                callback: (confirm) => {
                    if (confirm) {
                        this.deleteTodo(todoId)
                    } else {
                        return false
                    }
                }
            })
        },

        async updatedStatus(todo) {
            // console.log(todo.completed)
            // try {
            //     const res = this.$axios.patch(`/api/task/${todo.id}`, { completed: todo.completed })
            //     this.fetchClientsData()
            //     // console.log(res)
            // } catch (error) {
            //     console.log(error);
            // }
        },

        handleCurrentPageUpdate(currentPage) {
            this.currentPage = currentPage;
            this.fetchClientsData()
        },

        handleItemsPerPageUpdate(itemsPerPage) {
            this.itemsPerPage = itemsPerPage;
            this.fetchClientsData()
        },

        async fetchClientsData() {
            // try {
            //     this.loading = true
            //     const { data } = await this.$axios.get(
            //         `/api/tasks?page=${this.currentPage}&perPage=${this.itemsPerPage}&sortBy=${this.sortBy}&desc=${this.sortDesc}`
            //     );
            //     // console.log(this.$auth)

            //     this.todos = data.data.data;
            //     this.totalClients = data.data.total;

            // } catch (error) {
            //     console.log(error)
            // } finally {
            //     this.loading = false
            // }
        },

        openClientDialog() {
            this.addClientDialog = true;
        },

        closeSaveClientDialog() {
            this.addClientDialog = false
            this.todoData.title = ''
            this.todoData.description = ''
            this.todoData.completed = false
        },

        closeEditClientDialog(){
            this.clientIdToEdit = 0;
            this.todoData.title ='';
            this.todoData.description = '';
            this.todoData.completed = false;
            this.editClientDialog = false;
        },

        async saveClientData() {
            // try {
            //     this.loading = true
            //     await this.$axios.get('/sanctum/csrf-cookie')
            //     const res = await this.$axios.post('/api/tasks', this.todoData)
            //     // console.log(res)
            //     this.snackbarText = "Your task has been added."
            //     this.snackbar = true
            //     this.closeSaveClientDialog();
            //     await this.fetchClientsData();
            // } catch (error) {
            //     console.log(error)
            //     this.loading = false
            // } finally {
            //     this.loading = false
            // }
        },

        editTodoData(todo) {
            // this.clientIdToEdit = todo.id;
            // this.todoData.title = todo.title;
            // this.todoData.description = todo.description;
            // this.todoData.completed = todo.completed
            // this.editClientDialog = true;
        },

        async updateClientData() {
            // const updatedData = {
            //     title: this.todoData.title,
            //     description: this.todoData.description,
            //     completed: this.todoData.completed
            // }
            // try {
            //     this.loading = true
            //     const res = this.$axios.put(`/api/task/${this.clientIdToEdit}`, updatedData)
            //     // console.log(res)
            //     this.snackbarText = "Your task has been updated."
            //     this.snackbar = true
            //     this.closeEditClientDialog();
            //     await this.fetchClientsData();
            // } catch (error) {
            //     console.log(error)
            // } finally {
            //     this.loading = false
            // }
        },

        async deleteTodo(todoId) {
            // this.clientIdToDelete = todoId
            // try {
            //     await this.$axios.delete(`/api/task/${todoId}`);
            //     this.snackbarText = "Your task has been deleted."
            //     this.snackbar = true
            //     this.fetchClientsData();
            // } catch (error) {
            //     console.log(error)
            // } finally {
            //     this.clientIdToDelete = null
            // }
        },
    },
}
</script>
  