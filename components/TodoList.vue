<template>
    <div class="todoCard d-flex flex-wrap my-8">
        <v-card width="250">
            <v-card-title class="text-h5">
                {{todoItem.title}}
            </v-card-title>

            <v-card-subtitle>{{todoItem.description}}</v-card-subtitle>

            <v-card-actions>
                <v-btn text>Edit</v-btn>
                <v-btn text @click="$emit('delete', index)">
                    Remove
                </v-btn>
            </v-card-actions>
        </v-card>
    </div>
</template>

<script>

export default {
    props: {
        todoItem: {
            id: null,
            description: '',
            todoCheckbox: false,
            todoIndex: ''
        }
    }
}
</script>

<style>
input {
    border: 2px white;
    color: white;
}

.todoList li {
    list-style: none;
    margin: 20px;
}
</style>