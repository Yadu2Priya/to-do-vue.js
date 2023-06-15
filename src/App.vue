<template>
  <div>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
  <i class="bi bi-journal-text text-white mr-2 logo-icon"></i>
    <!-- <h3 class="navbar-brand">To Do App</h3> -->
  <ul class="navbar-nav">
    <li class="nav-item" :class="{ active: activePage === 'MyTasks' }">
      <a class="nav-link clickable" @click="changePage('MyTasks')">My Tasks</a>
    </li>
    <li class="nav-item" :class="{ active: activePage === 'Archive' }">
      <a class="nav-link clickable" @click="changePage('Archive')">Archive</a>
    </li>
  </ul>
  <div class="d-flex align-items-center">
    <i class="bi bi-person-circle mr-2" style="color: grey;"></i>
    <span class="username text-white" style="margin-left: 10px;">{{ displayUsername }}</span> 
   </div>
</nav>

    <main class="container">
      <h3>To Do List</h3>
      <div class="input-container">
        <div class="row">
          <div class="col-md-7 mb-4">
            <input
              v-if="activePage === 'MyTasks'"
              v-model="newTaskText"
              type="text"
              class="form-control"
              placeholder="Add a new task"
            />
          </div>
          <div class="col-md-4">
            <button
              v-if="activePage === 'MyTasks'"
              @click="addTask"
              class="btn btn-primary"
            >
              Add
            </button>
            
          </div>
          <h4>{{ headline }}</h4>
          <div class="col-md-7 mb-2">
            <input
              v-model="searchValue"
              type="text"
              class="form-control"
              placeholder="Search tasks"
            />
          </div>
        </div>
      </div>
      <div class="row">
        <div class="col-md-4" v-for="task in filteredTasks" :key="task.id">
          <div class="card">
            <div class="card-body">
              <input
                class="nav-link clickable"
                type="checkbox"
                v-model="task.completed"
                @change="toggleTask(task)"
              />
              <span class="card-text">{{ task.title }}</span>
              <i
                v-if="activePage === 'Archive'"
                @click="deleteTaskFromArchive(task)"
                class="bi bi-trash-fill delete-icon"
              ></i>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>


<script>
import 'bootstrap-icons/font/bootstrap-icons.css';
import axios from "axios";

export default {
  data() {
    return {
      activePage: "MyTasks",
      username: "David Muller",
      tasks: [],
      archiveTasks: [],
      newTaskText: "",
      searchValue: "",
      screenWidth: window.innerWidth,
    };
  },
  computed: {
    headline() {
      if (this.activePage === "MyTasks") {
        return `${this.filteredTasks.length} Open Tasks`;
      } else if (this.activePage === "Archive") {
        return `${this.archiveTasks.length} Completed Tasks`;
      }
    },
    filteredTasks() {
      const query = this.searchValue.toLowerCase();
      if (this.activePage === "MyTasks") {
        return this.tasks.filter(
          (task) => !task.completed && task.title.toLowerCase().includes(query)
        );
      } else if (this.activePage === "Archive") {
        return this.archiveTasks.filter((task) =>
          task.title.toLowerCase().includes(query)
        );
      }
    },
    displayUsername() {
      if (this.screenWidth < 640) {
        return this.username
          .split(" ")
          .map((word) => word.charAt(0))
          .join("");
      } else {
        return this.username;
      }
    },
  },
  created() {
    this.fetchTasks();
    window.addEventListener("resize", this.handleResize);
  },
  destroyed() {
    window.removeEventListener("resize", this.handleResize);
  },
  methods: {
    fetchTasks() {
      const storedTasks = localStorage.getItem('tasks');
      const storedArchiveTasks = localStorage.getItem('archiveTasks');
      if (storedTasks) {
        this.tasks = JSON.parse(storedTasks);
      }
      if (storedArchiveTasks) {
        this.archiveTasks = JSON.parse(storedArchiveTasks);
      }
      if (!storedTasks || !storedArchiveTasks) {
        axios
          .get("http://jsonplaceholder.typicode.com/todos?userId=1")
          .then((response) => {
            this.tasks = response.data.filter((task) => !task.completed);
            this.archiveTasks = response.data.filter((task) => task.completed);
            localStorage.setItem('tasks', JSON.stringify(this.tasks));
            localStorage.setItem('archiveTasks', JSON.stringify(this.archiveTasks));
          })
          .catch((error) => {
            console.error(error);
          });
      }
    },
    addTask() {
      if (this.newTaskText.trim() !== "") {
        const newTask = {
          userId: 1,
          id: Date.now(),
          title: this.newTaskText.trim(),
          completed: false,
        };
        this.tasks.unshift(newTask);
        this.newTaskText = "";

        localStorage.setItem('tasks', JSON.stringify(this.tasks));
      }
    },
    deleteTaskFromArchive(task) {
      if (confirm("Are you sure you want to permanently delete this task from the archive?")) {
        this.archiveTasks = this.archiveTasks.filter((t) => t !== task);
        localStorage.setItem("archiveTasks", JSON.stringify(this.archiveTasks));
      }
    },
    toggleTask(task) {
      if (task.completed) {
        this.tasks = this.tasks.filter((t) => t !== task);
        this.archiveTasks.unshift(task);
      } else {
        this.tasks.unshift(task);
        this.archiveTasks = this.archiveTasks.filter((t) => t !== task);
      }

      localStorage.setItem('tasks', JSON.stringify(this.tasks));
      localStorage.setItem('archiveTasks', JSON.stringify(this.archiveTasks));
    },
    changePage(page) {
      this.activePage = page;
    },
    handleResize() {
      this.screenWidth = window.innerWidth;
    },
  },
};
</script>




