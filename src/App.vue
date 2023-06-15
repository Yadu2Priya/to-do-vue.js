<template>
  <div>
    <!-- Nav bar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <i class="bi bi-journal-text text-white mr-2 logo-icon"></i>
      <ul class="navbar-nav">
        <!--highlighting Active page  -->
        <li class="nav-item" :class="{ active: activePage === 'MyTasks' }">
          <a class="nav-link clickable" @click="changePage('MyTasks')"
            >My Tasks</a
          >
        </li>
        <li class="nav-item" :class="{ active: activePage === 'Archive' }">
          <a class="nav-link clickable" @click="changePage('Archive')"
            >Archive</a
          >
        </li>
      </ul>
      <!-- User icon nd name -->
      <div class="d-flex align-items-center">
        <i class="bi bi-person-circle mr-2" style="color: grey"></i>
        <span class="username text-white" style="margin-left: 10px">{{
          displayUsername
        }}</span>
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
          <!-- Search input -->
          <div class="col-md-7 mb-2">
            <input
              v-model="searchValue"
              type="text"
              class="form-control"
              placeholder="Search task"
            />
          </div>
        </div>
      </div>
      <!-- Task cards -->
      <div class="row">
        <div v-if="searchTasks.length > 0" class="row">
          <div class="col-md-4" v-for="task in searchTasks" :key="task.id">
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
                  style="color: grey; margin-left: 10px"
                ></i>
              </div>
            </div>
          </div>
        </div>
        <div v-else style="margin-left:10px;">No tasks available</div>
      </div>
    </main>
  </div>
</template>

<script>
import "bootstrap-icons/font/bootstrap-icons.css";
import axios from "axios"; //Axios library for HTTP requests

export default {
  data() {
    return {
      activePage: "MyTasks", // Current active page
      username: "Priya Priya",
      tasks: [], // Array to store tasks
      archiveTasks: [], // Array to store archived tasks
      newTaskText: "",
      searchValue: "",
      screenWidth: window.innerWidth,
    };
  },
  computed: {
    headline() {
      if (this.activePage === "MyTasks") {
        return `${this.searchTasks.length} Open Tasks`; // Display the number of open tasks
      } else if (this.activePage === "Archive") {
        return `${this.archiveTasks.length} Completed Tasks`; // Display the number of completed tasks
      }
    },
    searchTasks() {
      const query = this.searchValue.toLowerCase();
      if (this.activePage === "MyTasks") {
        return this.tasks.filter(
          (task) => !task.completed && task.title.toLowerCase().includes(query)
        ); // Filter open tasks that match the search query
      } else if (this.activePage === "Archive") {
        return this.archiveTasks.filter((task) =>
          task.title.toLowerCase().includes(query)
        ); // Filter completed tasks that match the search query
      }
    },
    displayUsername() {
      if (this.screenWidth < 640) {
        return this.username
          .split(" ")
          .map((word) => word.charAt(0))
          .join(""); // display initial of username if the screen width is less than 640px
      } else {
        return this.username;
      }
    },
  },
  created() {
    this.fetchTasks(); // Fetch tasks when the component is created
    window.addEventListener("resize", this.handleScreenSize); // Add resize event listener to handle screen width changes
  },
  destroyed() {
    window.removeEventListener("resize", this.handleScreenSize); // Remove resize event listener when the component is destroyed
  },
  methods: {
    fetchTasks() {
      const storedTasks = localStorage.getItem("tasks");
      const storedArchiveTasks = localStorage.getItem("archiveTasks");
      if (storedTasks) {
        this.tasks = JSON.parse(storedTasks); // Retrieve tasks from localStorage if available
      }
      if (storedArchiveTasks) {
        this.archiveTasks = JSON.parse(storedArchiveTasks); // Retrieve archived tasks from localStorage if available
      }
      if (!storedTasks || !storedArchiveTasks) {
        axios
          .get("http://jsonplaceholder.typicode.com/todos?userId=1")
          .then((response) => {
            this.tasks = response.data.filter((task) => !task.completed); // Set tasks to open tasks from the API response
            this.archiveTasks = response.data.filter((task) => task.completed); // Set archiveTasks to completed tasks from the API response
            localStorage.setItem("tasks", JSON.stringify(this.tasks)); // Store tasks in localStorage
            localStorage.setItem(
              "archiveTasks",
              JSON.stringify(this.archiveTasks)
            );
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
        this.tasks.unshift(newTask); // Add new task to the tasks array
        this.newTaskText = ""; // Clear new task input

        localStorage.setItem("tasks", JSON.stringify(this.tasks)); // Update tasks in localStorage
      }
    },
    deleteTaskFromArchive(task) {
      if (
        confirm(
          "Are you sure you want to permanently delete this task from the archive?"
        )
      ) {
        this.archiveTasks = this.archiveTasks.filter((t) => t !== task);
        localStorage.setItem("archiveTasks", JSON.stringify(this.archiveTasks)); // Update archived tasks in localStorage
      }
    },
    toggleTask(task) {
      if (task.completed) {
        this.tasks = this.tasks.filter((t) => t !== task); // Remove the task from the tasks array
        this.archiveTasks.unshift(task); // Add the task to the beginning of the archiveTasks array
      } else {
        this.tasks.unshift(task); // Add the task to the beginning of the tasks array
        this.archiveTasks = this.archiveTasks.filter((t) => t !== task); // Remove the task from the archiveTasks array
      }

      localStorage.setItem("tasks", JSON.stringify(this.tasks)); // Update tasks
      localStorage.setItem("archiveTasks", JSON.stringify(this.archiveTasks));
    },
    changePage(page) {
      this.activePage = page; // Change the active page
    },
    handleScreenSize() {
      this.screenWidth = window.innerWidth;
    },
  },
};
</script>
