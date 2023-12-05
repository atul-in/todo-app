<template>
  <div class="main-container">

    <h1 class="text-center">To-do List App</h1>

    <!-- Add todo Dialog -->
    <v-dialog v-model="isDialogOpen" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">Create New To-Do</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field v-model="todoData.title" label="Title" required></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="todoData.description" label="Description" required></v-text-field>
              </v-col>
              <v-col>
                <v-checkbox v-model="todoData.completed" label="Completed"></v-checkbox>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="isDialogOpen = false">
            Close
          </v-btn>
          <v-btn color="blue darken-1" :loading="loading" text @click="saveTodo">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit todo Dialog -->

    <v-dialog v-model="editDialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="text-h5">Edit To-Do</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field v-model="todoData.title" label="Title" required></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field v-model="todoData.description" label="Description" required></v-text-field>
              </v-col>
              <v-col>
                <v-checkbox v-model="todoData.completed" label="Mark as completed"></v-checkbox>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="editDialog = false">
            Close
          </v-btn>
          <v-btn color="blue darken-1" :loading="loading" text @click="saveEditedTodo">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- todo Table -->


    <div class="d-flex align-center absolute justify-center">
      <v-container>
        <v-data-table :loading="loading" :headers="headers" :items="todos" class="elevation-1"
          :footer-props="{ 'items-per-page-options': [5, 10, 50, 100, -1] }" @update:page="handlePageUpdate"
          @update:items-per-page="handlePerPageUpdate" :server-items-length="totalTodos">
          <template #item.index="{ item, index }">{{ index + 1 }}</template>
          <template #item.created_at="{ item }">{{ $dayjs(created_at).format('DD/MM/YYYY') }}</template>
          <template #item.completed="{ item }"><v-checkbox v-model="item.completed"
              @change="updatedStatus(item)"></v-checkbox></template>
          <template v-slot:item.action_buttons="{ item }">

            <div class="d-flex">
              <v-btn :loading="item.loading" elevation="0" icon color="green" @click="editTodoData(item)">
                <v-icon dark>mdi-pencil</v-icon>
              </v-btn>

              <v-btn v-if="item.id!=itemIdToDelete" :loading="item.loading" elevation="0" icon color="red" @click="deleteTodo(item.id)">
                <v-icon dark>mdi-delete</v-icon>
              </v-btn>
              <v-progress-circular v-else indeterminate color="primary"></v-progress-circular>
            </div>
          </template>
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
import { eventBus } from '../eventBus';

export default {

  data() {
    return {
      itemIdToDelete:null,
      isDialogOpen: false,
      snackbar: false,
      snackbarText: '',
      editTodoId: '',
      todoData: {
        title: "",
        description: "",
        completed: false,
        user_id: this.$auth.user.data.id,
      },
      sortBy: "created_at",
      sortDesc: true,
      page: 1,
      perPage: 10,
      loading: false,
      totalTodos: 0,
      headers: [
        { text: "S.no.", value: "index", sortable: true },
        { text: "Title", value: "title", sortable: true },
        { text: "Description", value: "description", sortable: true },
        { text: "Completed", value: "completed", sortable: true },
        { text: "Created At", value: "created_at", sortable: true },
        { text: "Action", value: "action_buttons", sortable: false },
      ],

      progressbar: false,

      editDialog: false,
      todoEditTitle: '',
      todoEditDescription: '',
      currentlyEdit: null,
    }
  },

  created() {
    eventBus.$on("open-todo-modal", this.openTodoModal)
    eventBus.$on("logout-user", this.logout)
  },

  mounted() {
    this.fetchTodoData()
  },

  methods: {

    async updatedStatus(todo) {
      console.log(todo.completed)
      try {
        const res = this.$axios.patch(`/api/task/${todo.id}`, { completed: todo.completed })
        this.fetchTodoData()
        // console.log(res)
      } catch (error) {
        console.log(error);
      }
    },

    handlePageUpdate(page) {
      this.page = page;
      this.fetchTodoData()
    },

    handlePerPageUpdate(perPage) {
      this.perPage = perPage;
      this.fetchTodoData()
    },

    async fetchTodoData() {
      try {
        this.loading = true
        const { data } = await this.$axios.get(
          `/api/tasks?page=${this.page}&perPage=${this.perPage}&sortBy=${this.sortBy}&desc=${this.sortDesc}`
        );

        console.log(this.$auth)
        this.todos = data.data.data;
        this.totalTodos = data.data.total;

      } catch (error) {
        console.log(error)
      } finally {
        this.loading = false
      }
    },

    openTodoModal() {
      this.isDialogOpen = true;
    },

    async saveTodo() {
      try {
        this.loading = true
        await this.$axios.get('/sanctum/csrf-cookie')
        const res = await this.$axios.post('/api/tasks', this.todoData)
        // console.log(res)
        this.isDialogOpen = false
        this.snackbarText = "Your task has been added."
        this.snackbar = true
        this.fetchTodoData();
        this.editTodoId = ''
        this.todoData.title = ''
        this.this.todoData.description = ''
        this.this.todoData.completed = false
        this.loading = false
      } catch (error) {
        console.log(error)
      }
    },

    editTodoData(todo) {
      console.log(todo)
      this.editTodoId = todo.id;
      this.todoData.title = todo.title;
      this.todoData.description = todo.description;
      this.todoData.completed = todo.completed
      this.editDialog = true;
    },

    async saveEditedTodo() {
      const updatedData = {
        title: this.todoData.title,
        description: this.todoData.description,
        completed: this.todoData.completed
      }
      try {
        this.loading = true
        const res = this.$axios.put(`/api/task/${this.editTodoId}`, updatedData)
        console.log(res)
        this.editDialog = false
        this.snackbarText = "Your task has been updated."
        this.snackbar = true
        this.fetchTodoData();
        this.loading = false
      } catch (error) {
        console.log(error)
      }
    },

    async deleteTodo(todoId) {
      this.itemIdToDelete=todoId
      try {
        await this.$axios.delete(`/api/task/${todoId}`);
        this.snackbarText = "Your task has been deleted."
        this.snackbar = true
        this.fetchTodoData();
      } catch (error) {
        console.log(error)
      } finally{
        this.itemIdToDelete=null
      }
    },

    async logout() {
      try {
        await this.$auth.logout()
        this.$router.push('/login')
      } catch (error) {
        console.log(error)
      }
    },
  },
}
</script>
