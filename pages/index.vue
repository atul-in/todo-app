<template>
  <div class="main-container">
    <h1>To-do List App</h1>

    <v-btn class="my-5" @click.native.stop="dialog = true">+ New Todo</v-btn>


    <!-- Add todo Dialog -->
    <div>
      <v-dialog v-model="dialog" persistent max-width="600px">
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
            <v-btn color="blue darken-1" text @click="dialog = false">
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






    <!-- <div class="todoForm my-5">
      <form @submit.prevent="addTodo">
        <input type="text" v-model="newTodoItem.title" placeholder="Enter To-do Title?">
        <input type="text" v-model="newTodoItem.description" placeholder="Add Description">
        <v-btn type="submit" @click="snackbar = true">Add To-do</v-btn>
        <v-snackbar v-model="snackbar">
          {{ snackbarMsg }}
          <template v-slot:actions>
            <v-btn color="red" variant="text" @click="snackbar = false">
              Close
            </v-btn>
          </template>
        </v-snackbar>
      </form>
    </div> -->
    <!-- <ul v-for="(todo, index) in todos" :key="index">
      <TodoList :todoItem="todo" @delete="deleteTodo(index)" />
    </ul> -->
  </div>
</template>

<script>
// import TodoList from '../components/TodoList.vue';

export default {
//   components: {
//     TodoList,
//   },

  data() {
    return {
      dialog: false,

      todoId: null,
      todoDescription: '',
      todoCheckbox: false,

      editDialog: false,
      todoEditDescription: '',
      currentlyEdit: null,

      todos: [
        {
          todoId: 1,
          todoDescription: "asfdnjawd",
          todoCheckbox: true
        }
      ]
    }
  },

  methods: {
    addTodo() {
      this.todos.push({
        todoId: this.todos.length + 1,
        todoDescription: this.todoDescription,
        todoCheckbox: this.todoCheckbox
      })
      this.todoDescription = ''
      this.todoCheckbox = false
      this.dialog = false
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
    }
  },
}
</script>
