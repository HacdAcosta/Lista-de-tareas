<template>
  <div class="my-component-container"> 
  <div id="app">
    <h1>Lista de Tareas</h1>
    <div class="task-input">
      <input type="text" v-model="newTask.text" placeholder="Agrega una nueva tarea" @keyup.enter="addTask" />
      <input type="text" v-model="newTask.description" placeholder="Descripción de la tarea" @keyup.enter="addTask" />
      <input type="datetime-local" v-model="newTask.dueDate" :min="getCurrentDateTime" />
      <button @click="addTask">Agregar</button>
    </div>
    <div class="tasks">
      <table>
        <thead>
          <tr>
            <th>Tarea</th>
            <th>Descripción</th>
            <th>Creada</th>
            <th>Vence</th>
            <th>Tiempo Restante</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(task, index) in tasks" :key="index">
            <td :class="{ 'completed': task.completed, 'incomplete': !task.completed }">
              <input v-if="task.editing" type="text" v-model="task.text" />
              <span v-else>{{ task.text }}</span>
            </td>
            <td>
              <input v-if="task.editing" type="text" v-model="task.description" />
              <span v-else>{{ task.description }}</span>
            </td>
            <td>{{ formatDateTime(task.createdAt) }}</td>
            <td>{{ formatDateTime(task.dueDate) }}</td>
            <td>
              <span v-if="timeRemaining(task.dueDate).days > 0">{{ timeRemaining(task.dueDate).days }} días</span>
              <span v-else-if="timeRemaining(task.dueDate).hours > 0">{{ timeRemaining(task.dueDate).hours }} horas</span>
              <span v-else-if="timeRemaining(task.dueDate).minutes > 0">{{ timeRemaining(task.dueDate).minutes }} minutos</span>
              <span v-else>Vencida</span>
            </td>
            <td>
              <button v-if="!task.editing" @click="editTask(index)">Editar</button>
              <button v-if="task.editing" @click="saveTask(index)">Guardar</button>
              <button @click="removeTask(index)">Eliminar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
  </div>
</template>

<script>
import Swal from 'sweetalert2'
import { get, set } from 'idb-keyval'

export default {
  name: 'App',
  data() {
    return {
      newTask: {
        text: '',
        description: '',
        dueDate: new Date().toISOString().slice(0, 16),
        createdAt: new Date(),
        completed: false
      },
      tasks: []
    }
  },
  created() {
    this.loadTasks()
  },
  computed: {
    getCurrentDateTime() {
      return new Date().toISOString().slice(0, 16)
    }
  },
  methods: {
    addTask() {
      if (this.newTask.text.trim() !== '' && this.newTask.dueDate >= this.getCurrentDateTime) {
        this.tasks.push({
          ...this.newTask,
          editing: false
        })
        this.newTask = {
          text: '',
          description: '',
          dueDate: new Date().toISOString().slice(0, 16),
          createdAt: new Date(),
          completed: false
        }
        this.saveTasks()
        Swal.fire({
          title: '¡Tarea creada!',
          text: 'La nueva tarea ha sido agregada a la lista.',
          icon: 'success',
          confirmButtonText: 'Aceptar'
        })
      } else {
        Swal.fire({
          title: 'Error',
          text: 'Por favor, ingresa una tarea válida y una fecha de vencimiento que sea posterior a la fecha y hora actuales.',
          icon: 'error',
          confirmButtonText: 'Aceptar'
        })
      }
    },
    removeTask(index) {
      Swal.fire({
        title: '¿Estás seguro?',
        text: "¡No podrás revertir esto!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: '¡Sí, elimínala!'
      }).then((result) => {
        if (result.isConfirmed) {
          this.tasks.splice(index, 1)
          this.saveTasks()
          Swal.fire(
            '¡Eliminada!',
            'La tarea ha sido eliminada.',
            'success'
          )
        }
      })
    },
    editTask(index) {
      this.tasks[index].editing = true
    },
    saveTask(index) {
      this.tasks[index].editing = false
      this.saveTasks()
      Swal.fire({
        title: '¡Tarea actualizada!',
        text: 'La tarea ha sido actualizada.',
        icon: 'success',
        confirmButtonText: 'Aceptar'
      })
    },
    async saveTasks() {
      try {
        const userId = localStorage.getItem('userId') || this.generateUserId()
        await set(`tasks-${userId}`, this.tasks)
        localStorage.setItem('userId', userId)
      } catch (error) {
        console.error('Error al guardar las tareas:', error)
      }
    },
    async loadTasks() {
      try {
        const userId = localStorage.getItem('userId') || this.generateUserId()
        this.tasks = (await get(`tasks-${userId}`)) || []
        localStorage.setItem('userId', userId)
      } catch (error) {
        console.error('Error al cargar las tareas:', error)
      }
    },
    generateUserId() {
      return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
        var r = Math.random() * 16 | 0, v = c == 'x' ? r : (r & 0x3 | 0x8)
        return v.toString(16)
      })
    },
    formatDateTime(date) {
      return new Date(date).toLocaleString()
    },
    timeRemaining(dueDate) {
      const now = new Date()
      const due = new Date(dueDate)
      const diff = due - now
      const days = Math.floor(diff / (1000 * 60 * 60 * 24))
      const hours = Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60))
      const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60))
      return { days, hours, minutes }
    }
  }
}
</script>



  <style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
    
  }
  
  .task-input {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 20px;
  }
  
  .task-input input {
    padding: 8px 12px;
    font-size: 16px;
    border: 1px solid #ccc;
    border-radius: 4px;
    margin-right: 10px;
  }
  
  .task-input button {
    padding: 8px 16px;
    font-size: 16px;
    background-color: #00B8FF;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  
  table {
    width: 100%;
    max-width: 100%; 
    border-collapse: collapse;
    font-size: 16px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
    border: 2px  #00B8FF;
    border-radius: 15px;
    border-color: #00B8FF;
  }
  
  th, td {
    padding: 12px 6px;
    text-align: left;
    border-right: 1px solid #00B8FF;
    max-width: 200px; 
    white-space: nowrap; 
    overflow: hidden; 
    text-overflow: ellipsis;
  }
  
  th {
    background-color: #00B8FF;
    font-weight: bold;
    color: white;
  }
  
  tr:nth-child(even) {
    background-color: #f2f2f2;
  }
  
  .incomplete {
    color: #333;
  }
  
  .completed {
    text-decoration: line-through;
    color: #9E9E9E;
  }
  
  td input[type="checkbox"] {
    transform: scale(1.5); /* Ajusta el tamaño del checkbox */
    margin-right: 10px; /* Espacio entre el checkbox y el texto */
  }
  
  td input[type="checkbox"]:checked + label {
    color: #00B8FF; /* Cambia el color del texto al seleccionar el checkbox */
  }
  
  td input[type="checkbox"]:checked + label::after {
    content: " - Completado"; /* Agrega texto adicional al completar la tarea */
  }
  
  .completed {
    text-decoration: line-through;
    color: #9E9E9E;
  }
  
  button {
    padding: 6px 12px;
    font-size: 14px;
    background-color: #00B8FF;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    margin-left: 2px
  }
  
  button:hover {
    background-color: #005E82;
  }
  .incomplete {
    color: #333;
  }
  
  .completed {
    text-decoration: line-through;
    color: #9E9E9E;
  }

</style>

