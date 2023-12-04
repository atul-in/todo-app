<template>
  <div class="main-container">
    <h1 class="text-center">To-do List App</h1>

    <!-- <v-btn class="my-5" @click.native.stop="isDialogOpen = true">+ New Todo</v-btn> -->


    <!-- Add todo Dialog -->
    <div>
      <v-dialog v-model="isDialogOpen" persistent max-width="600px">
        <v-card>
          <v-card-title>
            <span class="text-h5">Create New To-Do</span>
          </v-card-title>
          <v-card-text>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field v-model="todoDescription" label="Description" required></v-text-field>
                </v-col>
                <v-col>
                  <v-checkbox v-model="todoCheckbox" label="Mark as completed"></v-checkbox>
                </v-col>
              </v-row>
            </v-container>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" text @click="isDialogOpen = false">
              Close
            </v-btn>
            <v-btn color="blue darken-1" text @click="addTodo">
              Save
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </div>

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
                <v-text-field v-model="todoEditDescription" label="Description" required></v-text-field>
              </v-col>
              <v-col>
                <v-checkbox v-model="todoCheckbox" label="Mark as completed"></v-checkbox>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="editDialog = false">
            Close
          </v-btn>
          <v-btn color="blue darken-1" text @click="saveEditedTodo">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


    <!-- todo Table -->
    <!-- <div class="d-flex align-center absolute justify-center">
      <v-table class="my-5">
        <thead>
          <tr>
            <th class="text-left" width="100">
              S. No.
            </th>
            <th class="text-left" width="300">
              Description
            </th>
            <th class="text-left" width="200">
              Completed
            </th>
            <th class="text-left" width="200">
              Actions
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="todo.todoId">
            <td>{{ index + 1 }}</td>
            <td>{{ todo.todoDescription }}</td>
            <td><v-checkbox v-model="todo.todoCheckbox"></v-checkbox></td>
            <td>
              <v-card-actions>
                <v-btn class="blue--text" @click="editTodoDialog(todo)" text>Edit</v-btn>
                <v-btn class="red--text" @click="deleteTodo(todo.todoId)" text>Remove</v-btn>
              </v-card-actions>
            </td>
          </tr>
        </tbody>
      </v-table>
    </div> -->

    <div class="d-flex align-center absolute justify-center">
      <v-data-table
        :loading="loading"
        :headers="headers"
        :items="todos"
        class="elevation-1"
        :server-items-length="totalTodos"
      >
      <template #item.index="{item, index}">{{ index+1 }}</template>
      <template #item.created_at="{ item }">{{ $dayjs(created_at).format('DD/MM/YYYY') }}</template>
      </v-data-table>
    </div>

    <v-btn class="red--text" @click.prevent="logout">Logout</v-btn>

  </div>
</template>

<script>
// import TodoList from '../components/TodoList.vue';
import { eventBus } from '../eventBus';

export default {
  //   components: {
  //     TodoList,
  //   },

  auth: false,

  data() {
    return {
      isDialogOpen: false,
      
      todoId: null,
      todoDescription: '',
      todoCheckbox: false,
      
      editDialog: false,
      todoEditDescription: '',
      currentlyEdit: null,
      
      loading: false,
      totalTodos:0,
      todos: [],
      headers:[
        {text:"S.no.", value:"index", sortable:false},
        {text:"Title", value:"title", sortable:false},
        {text:"Description", value:"description", sortable:false},
        {text:"Created At", value:"created_at", sortable:false},
        {text:"Action", value:"", sortable:false},
      ]
    }
  },

  created() {
    eventBus.$on("open-todo-modal", this.openTodoModal)
  },

  mounted(){
    this.fetchTodoData()
  },

  methods: {

    async fetchTodoData(){
      try {
        const {data} = await this.$axios.get('/api/tasks');
        this.todos = data;
        this.totalTodos = data.length;

        
      } catch (error) {
        console.log(error)
      }
    },

    openTodoModal() {
      this.isDialogOpen = true;
    },
    addTodo() {
      this.todos.push({
        todoId: this.todos.length + 1,
        todoDescription: this.todoDescription,
        todoCheckbox: this.todoCheckbox
      })
      this.todoDescription = ''
      this.todoCheckbox = false
      this.isDialogOpen = false
      alert("Your todo has been added successfully.")
    },

    editTodoDialog(todo) {
      this.currentlyEdit = todo.todoId;
      this.todoEditDescription = todo.todoDescription;
      this.todoCheckbox = todo.todoCheckbox;
      this.editDialog = true;
    },

    saveEditedTodo() {
      const editedTodoIndex = this.todos.findIndex(todo => todo.todoId === this.currentlyEdit);

      if (editedTodoIndex !== -1) {
        this.todos[editedTodoIndex].todoDescription = this.todoEditDescription;
        this.todos[editedTodoIndex].todoCheckbox = this.todoCheckbox;
        this.editDialog = false;
        alert("Your todo has been edited successfully.")
      }
    },

    deleteTodo(id) {
      const todoIndex = this.todos.indexOf(id)
      this.todos.splice(todoIndex, 1)
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
