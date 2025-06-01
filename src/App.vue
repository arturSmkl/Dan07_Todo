<script setup>
import {ref, watch } from 'vue'

// Task structure
const defaultTasks = ref({
  todo: ['Buy milk', 'Call mom'],
  doing: ['Write report'],
  done: ['Morning run']
})

const storedTasks = localStorage.getItem('tasks')
const tasks = ref(storedTasks ? JSON.parse(storedTasks) : defaultTasks)

// Save to localStorage whenever tasks change
watch(tasks, (newTasks) => {
  localStorage.setItem('tasks', JSON.stringify(newTasks))
}, { deep: true })

const columnTitles = {
  todo: 'To Be Done',
  doing: 'Doing',
  done: 'Done'
}

const statusKeys = ['todo', 'doing', 'done']

const dialog = ref(false)
const newTask = ref('')

function addTask() {
  if (newTask.value.trim()) {
    tasks.value.todo.push(newTask.value.trim())
    newTask.value = ''
    dialog.value = false
  }
}


// Move task left/right
function moveTask(currentStatus, index, direction) {
  const currentIndex = statusKeys.indexOf(currentStatus)
  const newStatus = statusKeys[currentIndex + direction]

  if (newStatus) {
    const task = tasks.value[currentStatus].splice(index, 1)[0]
    tasks.value[newStatus].push(task)
  }
}

// Delete task
function deleteTask(status, index) {
  tasks.value[status].splice(index, 1)
}
</script>

<template>
  <v-app>
    <v-container>
      <v-dialog v-model="dialog" persistent max-width="500">
        <v-card>
          <v-card-title>
            <span class="text-h6">Add New Task</span>
          </v-card-title>
          <v-card-text>
            <v-text-field
                label="Task Description"
                v-model="newTask"
                autofocus
                @keyup.enter="addTask"
            />
          </v-card-text>
          <v-card-actions>
            <v-spacer />
            <v-btn text @click="dialog = false">Cancel</v-btn>
            <v-btn color="primary" variant="tonal" @click="addTask">Add</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Header -->
      <v-row class="mb-4 mt-4" align="center" justify="space-between">
        <v-col cols="6">
          <h1 class="text-h5 font-weight-bold">TO-DO List</h1>
        </v-col>
        <v-col cols="6" class="text-right">
          <v-btn color="primary" @click="dialog = true">Add New Task</v-btn>
        </v-col>
      </v-row>

      <!-- Task Columns -->
      <v-row>
        <v-col cols="12" sm="4" v-for="(list, status) in tasks" :key="status">
          <v-card>
            <v-card-title class="text-h6 d-flex justify-space-between">
              {{ columnTitles[status] }}
              <v-chip variant="outlined" color="primary" class="ml-2">{{ list.length }}</v-chip>
            </v-card-title>
            <v-divider></v-divider>
            <v-card-text>
              <v-list>
                <v-list-item
                    min-height="50px"
                    v-for="(task, index) in list"
                    :key="`${status}-${index}`"
                    class="d-flex justify-space-between align-center task-item"
                    :class="{ todo: status === 'todo', doing: status === 'doing', done: status === 'done' }"
                >
                  <div>{{ task }}</div>
                  <div class="task-controls">
                    <v-icon
                        color="primary"
                        class="mr-2"
                        v-if="status !== 'todo'"
                        @click="moveTask(status, index, -1)"
                        icon="mdi-arrow-left"
                    />

                    <v-icon
                        color="primary"
                        class="mr-4"
                        v-if="status !== 'done'"
                        @click="moveTask(status, index, 1)"
                        icon="mdi-arrow-right"
                    />

                    <v-icon
                        color="red"
                        @click="deleteTask(status, index)"
                        icon="mdi-delete"
                    />
                  </div>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
    <v-footer app border>
      <v-container class="text-center pa-0">
        <span class="text-caption">© 2025 Artur Smýkal</span>
      </v-container>
    </v-footer>
  </v-app>
</template>

<style scoped>
.task-item {
  transition: background-color 0.3s;
  position: relative;
}

.task-controls {
  display: none;
  align-items: center;
}

.task-item:hover {
  background-color: #f4f4f4;
}

.task-item:hover .task-controls {
  display: flex;
}

.todo {
  border-left: #104483 2px solid;
}

.doing {
  border-left: #1f88fd 2px solid;
}

.done {
  border-left: #f4f4f4 2px solid;
}

</style>
