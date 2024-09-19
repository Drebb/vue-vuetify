<template>
  <v-container>
    <v-form @submit.prevent="addTodo">
      <v-row>
        <v-col cols="12" md="8">
          <v-text-field v-model="newTodo" label="Add a new task" outlined clearable solo
            placeholder="e.g., Finish the project" :counter="30" maxlength="30" class="input-field" color="primary" />
        </v-col>
        <v-col cols="12" md="4">
          <v-btn type="submit" color="primary" block class="add-task-button" elevation="2">
            <v-icon left>mdi-plus</v-icon> Add Task
          </v-btn>
        </v-col>
      </v-row>
    </v-form>

    <v-alert v-if="error" type="error" class="mt-4" dismissible>
      {{ error }}
    </v-alert>

    <v-row class="my-4" dense>
      <v-col>
        <v-chip-group v-model="activeCategory" active-class="primary--text" column>
          <v-chip v-for="category in categories" :key="category" :value="category" class="ma-1"
            :color="activeCategory === category ? 'primary' : 'grey lighten-2'" label elevation="1">
            {{ category }}
          </v-chip>
        </v-chip-group>
      </v-col>
    </v-row>

    <v-row>
      <v-col v-for="todo in filteredTodos" :key="todo.id" cols="12" sm="6" md="4" lg="4">
        <v-card class="mb-4" outlined elevation="2">
          <v-card-title>
            <v-row align="center" justify="space-between" class="w-100">
              <v-col>
                <span :class="{ 'text-decoration-line-through': todo.completed }" @click="toggleExpand(todo.id)"
                  class="todo-title">
                  {{ todo.expanded ? todo.text : (todo.text.length > 15 ? todo.text.slice(0, 15) + '...' : todo.text) }}
                </span>
              </v-col>
            </v-row>
          </v-card-title>

          <v-card-subtitle v-if="activeCategory === 'All Tasks'">
            <span class="text-grey--text">{{ todo.category }} Category</span>
          </v-card-subtitle>

          <v-card-actions class="justify-end">
            <v-btn v-if="todo.category === 'Tasks'" @click="moveToInProgress(todo.id)" color="primary" icon
              elevation="1">
              <v-icon>mdi-play</v-icon>
            </v-btn>
            <v-btn v-if="todo.category === 'In Progress'" @click="moveToFinished(todo.id)" color="success" icon
              elevation="1">
              <v-icon>mdi-check</v-icon>
            </v-btn>
            <v-btn v-if="todo.category === 'Finished'" @click="deleteTodo(todo.id)" color="error" icon elevation="1">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
            <v-btn v-if="todo.category !== 'Tasks'" @click="moveBack(todo.id)" color="warning" icon elevation="1">
              <v-icon>mdi-arrow-left</v-icon>
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <v-alert v-if="filteredTodos.length === 0" type="info" class="mt-4" text outlined>
      No tasks in {{ activeCategory }}.
    </v-alert>
  </v-container>
</template>

<script setup>
import { ref, computed, watch } from 'vue';

const escapeHTML = (text) => {
  if (/&|<|>|"|'|{|}|;|%|\.\.\/|\.\.$/g.test(text)) {
    return null;
  }
  return text.trim();
};

const newTodo = ref('');
const todos = ref([]);
const activeCategory = ref('All Tasks');
const error = ref('');

const categories = ['All Tasks', 'Tasks', 'In Progress', 'Finished'];
const categoryOrder = {
  Tasks: 1,
  'In Progress': 2,
  Finished: 3,
};

watch(activeCategory, (newCategory) => {
  if (categories.includes(newCategory)) {
    activeCategory.value = newCategory;
  } else {
    activeCategory.value = 'All Tasks';
  }
});

const filteredTodos = computed(() => {
  if (activeCategory.value === 'All Tasks') {
    return todos.value.slice().sort((a, b) => categoryOrder[a.category] - categoryOrder[b.category]);
  }
  return todos.value.filter(todo => todo.category === activeCategory.value);
});

const isDuplicateTask = (text) => {
  return todos.value.some(todo => todo.text.toLowerCase() === text.toLowerCase());
};

const addTodo = () => {
  const trimmedInput = newTodo.value.trim();

  if (trimmedInput === '') {
    error.value = 'Please enter a task.';
    setTimeout(() => { error.value = ''; }, 3000);
    return;
  }
  if (trimmedInput.length > 30) {
    error.value = 'Task cannot exceed 30 characters.';
    setTimeout(() => { error.value = ''; }, 3000);
    return;
  }
  if (isDuplicateTask(trimmedInput)) {
    error.value = 'This task already exists.';
    setTimeout(() => { error.value = ''; }, 3000);
    return;
  }

  const sanitizedInput = escapeHTML(trimmedInput);

  if (sanitizedInput === null) {
    error.value = 'Scripting is not allowed.';
    setTimeout(() => { error.value = ''; }, 3000);
    return;
  }

  const id = Date.now();
  todos.value.push({ id, text: sanitizedInput, completed: false, category: 'Tasks', expanded: false });
  newTodo.value = '';
  error.value = '';
};

const toggleExpand = (id) => {
  const task = todos.value.find(todo => todo.id === id);
  if (task) task.expanded = !task.expanded;
};

const moveToInProgress = (id) => {
  const task = todos.value.find(todo => todo.id === id);
  if (task) task.category = 'In Progress';
};

const moveToFinished = (id) => {
  const task = todos.value.find(todo => todo.id === id);
  if (task) task.category = 'Finished';
};

const moveBack = (id) => {
  const task = todos.value.find(todo => todo.id === id);
  if (task) {
    if (task.category === 'Finished') {
      task.category = 'In Progress';
    } else if (task.category === 'In Progress') {
      task.category = 'Tasks';
    }
  }
};

const deleteTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
};

</script>

<style scoped>
.input-field {
  height: 56px;
}

.add-task-button {
  height: 56px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.todo-title {
  cursor: pointer;
  font-weight: bold;
  font-size: 1.1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.todo-title:hover {
  text-decoration: underline;
}

.text-decoration-line-through {
  text-decoration: line-through;
  color: grey;
}

.v-card {
  transition: box-shadow 0.3s ease;
  padding: 16px;
  border-radius: 8px;
}

.v-card:hover {
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.v-card-title {
  font-size: 1.2rem;
}

.v-card-subtitle {
  font-size: 0.9rem;
}
</style>
