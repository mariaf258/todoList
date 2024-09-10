<script setup lang="ts">
import TheWelcome from '../components/TheWelcome.vue'
import { ref, nextTick, onMounted, watch } from 'vue'

const tasks = ref([{ titulo: '', descripcion: 'Ejemplo', completed: false }])
const finalizedTasks = ref([])
const trashedTasks = ref([])
const activeView = ref('inicio')
const isMenuVisible = ref(false)

const toggleMenu = () => {
  isMenuVisible.value = !isMenuVisible.value
}

const handleClick = (item) => {
  if (item === 'inicio') {
    activeView.value = 'inicios'
  } else if (item === 'papelera') {
    activeView.value = 'papelera'
  }
}

function loadTasksFromLocalStorage() {
  const storedTasks = localStorage.getItem('tasks')
  if (storedTasks) {
    tasks.value = JSON.parse(storedTasks)
  }

  const storedFinalizedTasks = localStorage.getItem('finalizedTasks')
  if (storedFinalizedTasks) {
    finalizedTasks.value = JSON.parse(storedFinalizedTasks)
  }

  const storedTrashedTasks = localStorage.getItem('trashedTasks')
  if (storedTrashedTasks) {
    trashedTasks.value = JSON.parse(storedTrashedTasks)
  }
}

onMounted(() => {
  loadTasksFromLocalStorage()
})

watch(
  [tasks, finalizedTasks, trashedTasks],
  () => {
    localStorage.setItem('tasks', JSON.stringify(tasks.value))
    localStorage.setItem('finalizedTasks', JSON.stringify(finalizedTasks.value))
    localStorage.setItem('trashedTasks', JSON.stringify(trashedTasks.value))
  },
  { deep: true }
)

function newElement() {
  const input = document.getElementById('myInput') as HTMLInputElement
  const taskText = input.value.trim()
  if (taskText) {
    tasks.value.push({ titulo: '', descripcion: taskText, completed: false })
    input.value = ''
    nextTick(() => showToast(tasks.value.length - 1))
  }
}

function showToast(index: number) {
  const toastEl = document.getElementById(`toast-${index}`)
  if (toastEl) {
    const toast = new bootstrap.Toast(toastEl)
    toast.show()
  }
}

function removeTask(index: number) {
  trashedTasks.value.push(tasks.value[index])
  tasks.value.splice(index, 1)
}

function markAsCompleted(task: any) {
  task.completed = !task.completed
  if (task.completed) {
    finalizedTasks.value.push(task)
  } else {
    finalizedTasks.value = finalizedTasks.value.filter((t) => t !== task)
  }
}
</script>

<template>
  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
    integrity="sha384-QWTKZyjpPEjISv5WaRU9OFeRpok6YctnYmDr5pNlyT2bRjXh0JMhjY6hW+ALEwIH"
    crossorigin="anonymous"
  />
  <div class="page-wrapper">
    <div class="">
      <div>
        <div v-if="isMenuVisible" class="menu">
          <ul>
            <li @click="handleClick('inicio')">Inicio</li>
            <hr />
            <li @click="handleClick('papelera')">Papelera</li>
          </ul>
        </div>
      </div>
      <header>
        <button @click="toggleMenu" class="menu-button">☰</button>
        <h1>LISTA DE COSAS POR HACER</h1>
        <div class="btn-group mb-3" role="group" aria-label="Menú"></div>
      </header>

      <div class="input-group mb-3">
        <input
          type="text"
          id="myInput"
          class="form-control"
          placeholder="Ingrese Tareas..."
          aria-label="Title"
        />
        <button @click="newElement" class="btn btn-primary addBtn">Agregar</button>
      </div>

      <main>
        <div v-if="activeView === 'inicio'" class="row">
          <!-- <pre>{{ tasks }}</pre> -->
          <div
            v-for="(task, index) in tasks"
            :key="index"
            :id="`toast-${index}`"
            class="col-md-4 mb-3"
          >
            <div
              class="toast show"
              role="alert"
              aria-live="assertive"
              aria-atomic="true"
              data-bs-autohide="false"
              :class="{ 'opacity-50': task.completed }"
            >
              <div class="toast-header">
                <strong class="me-auto">{{ task.titulo }}</strong>
                <button
                  type="button"
                  class="btn btn-sm btn-danger ms-2"
                  data-bs-dismiss="toast"
                  @click="removeTask(index)"
                  aria-label="Close"
                >
                  X
                </button>
                <button
                  type="button"
                  class="btn btn-sm btn-success ms-2"
                  @click="markAsCompleted(task)"
                  aria-label="Mark as completed"
                >
                  ✓
                </button>
              </div>
              <div class="toast-body">{{ task.descripcion }}</div>
            </div>
          </div>
        </div>
        <div v-if="activeView === 'showFinalizadas'" class="row">
          <div v-for="(task, index) in finalizedTasks" :key="index" class="col-md-4 mb-3">
            <div class="alert alert-success">
              <strong>{{ task.titulo }}</strong
              >: {{ task.descripcion }}
            </div>
          </div>
        </div>

        <div v-if="activeView === 'papelera'" class="row">
          <div v-for="(task, index) in trashedTasks" :key="index" class="col-md-4 mb-3">
            <div class="alert alert-danger">
              <strong>{{ task.titulo }}</strong
              >: {{ task.descripcion }}
            </div>
          </div>
        </div>
      </main>
    </div>
    <footer></footer>
  </div>
</template>

<style scoped>
.page-wrapper {
  background-color: #eaeaea;
  min-height: 100vh;
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 0 auto;
  margin: 0;
}

.container {
  text-align: center;
  width: 400vh;
  margin: 0 auto;
  background-color: #eaeaea;
  height: 95vh;
}

header,
footer {
  background: linear-gradient(to right, #8d0ee2, #aa30ff, #e1b6ff);
  padding: 10px;
  width: 100;
  text-align: center;
}

header h1 {
  color: white;
  margin: 0;
  font-size: 2rem;
  text-align: center;
  text-shadow: 2px 2px 4px black;
}

header .btn-secondary {
  display: flex;
  position: absolute;
  padding: none;
  border-top: 0%;
  background-color: transparent;
  border-color: transparent;
  text-align: left;
}

.input-group {
  margin: 10px auto;
  border-color: #12b7a5;
  box-shadow: #12b7a5;
  width: 60%;
  align-items: center;
  max-width: 600px;
  display: flex;
}

.form-control {
  color: #3d3d3d;
  border-color: #12b7a5;
  box-shadow: 5px 5px 10px 2px #bdbdbd;
  align-items: center;
}

.form-control:focus {
  border-color: #12b7a5;
}

.addBtn {
  color: white;
  background-color: #12b7a5;
  border-color: #12b7a5;
  box-shadow: 5px 5px 10px 2px #bdbdbd;
}

.addBtn:hover {
  background-color: #0c9589;
  border-color: #0c9589;
}

.AddBtn {
  background-color: transparent;
  text-align: left;
  z-index: 1;
  position: absolute;
  font-size: 20px;
}

.btn-secondary {
  background-color: transparent;
  border-color: transparent;
  outline: none;
  box-shadow: none;
  color: white;
  font-size: 15px;
}

.btn-secondary:hover,
.btn-secondary:focus,
.btn-secondary:active {
  background-color: transparent;
  border-color: transparent;
  outline: none;
  box-shadow: none;
  color: white;
}

.btn-group {
  text-align: left;
  background-color: #981bed, 0.5;
  justify-content: space-between;
  display: flex;
  justify-content: flex-start;
}

.toast-header {
  max-width: 1280px;
  margin: 0 auto;
  background-color: #e5beff;
  font-weight: normal;
}

.toast {
  width: auto;
  max-width: 350px;
  margin: 0 auto;
  font-weight: normal;
  overflow-wrap: break-word;
  word-break: break-word;
}

.toast-body {
  background-color: #ffffff;
  padding: auto;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toast-show {
  background-color: #f0f0f0;
  padding: 10px;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toast-container {
  max-width: 600px;
  margin: 0 auto;
}

.close {
  background-color: #ff6b6b;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  padding: 5px;
}

.completed {
  background-color: #c3f6c3;
  text-decoration: line-through;
}

.complete {
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  font-size: 14px;
  padding: 5px;
}

#app {
  display: column;
  grid-template-columns: 100%;
}

body {
  display: column;
  place-items: center;
}

.row {
  margin-right: auto;
}

.toast-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  z-index: 10;
}

.btn-sm {
  font-size: 0.5rem;
}

.alert-danger {
  overflow-wrap: break-word;
  word-break: break-word;
}

.menu-button {
  font-size: 24px;
  cursor: pointer;
  display: flex;
  background: none;
  border: none;
  color: white;
  padding: 10px;
  flex-direction: column;
  gap: 4px;
  margin-right: auto;
}

.menu {
  position: absolute;
  top: 60px;
  left: 0;
  background: #981bed;
  border:
    1px solid #981bed,
    0.5;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.5);
  width: 150px;
  height: 120px;
  border-radius: 5px;
  z-index: 1;
}

.menu ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.menu li {
  padding: 10px;
  cursor: pointer;
  font-size: 14px;
  color: white;
}

.menu li:hover {
  background: #981bed;
}

hr {
  color: white;
}
</style>
