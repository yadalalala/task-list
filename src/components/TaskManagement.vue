<template>
  <v-container class="container">
    <h1 class="title">TODO LIST</h1>
    <v-row align="center">
      <v-col>
        <v-btn color="primary" @click="openAddDialog">Add Task</v-btn>
      </v-col>
      <v-col cols="4" class="d-flex justify-end mt-4">
        <v-select
          density="compact"
          v-model="filter"
          :items="filterOptions"
          outlined
        ></v-select>
      </v-col>
    </v-row>

    <div class="task-list">
      <div v-for="task in filteredTasks" :key="task.id" class="task-item">
        <v-row class="pt-5">
          <input
            type="checkbox"
            v-model="task.completed"
            class="task-checkbox"
          />
          <div :class="{ completed: task.completed }" class="task-title">
            {{ task.title }}
          </div>

          <v-btn icon="mdi-eye" variant="text" @click="viewTask(task)"> </v-btn>
          <v-btn icon="mdi-delete" variant="text" @click="showAlert(task)">
          </v-btn>
          <v-btn icon="mdi-pencil" variant="text" @click="editTask(task)">
          </v-btn>
        </v-row>
        <span class="task-date">{{ task.timestamp }}</span>
      </div>
    </div>

    <!--  View-->
    <v-dialog v-model="viewDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Detail</span>
        </v-card-title>
        <v-card-text>
          <div><strong>Title:</strong> {{ currentTask?.title }}</div>
          <div>
            <strong>Status:</strong>
            {{ currentTask?.completed ? "Completed" : "Incomplete" }}
          </div>
          <div><strong>Timestamp:</strong> {{ currentTask?.timestamp }}</div>
        </v-card-text>
        <v-card-actions>
          <v-btn color="blue darken-1" text @click="closeViewDialog"
            >Close</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!--Add -->
    <v-dialog v-model="addDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Add New Task</span>
        </v-card-title>
        <v-card-text>
          <v-text-field
            label="Task Title"
            v-model="newTask.title"
            required
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn color="blue darken-1" text @click="closeAddDialog"
            >Cancel</v-btn
          >
          <v-btn color="blue darken-1" text @click="saveTask">Add</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Edit -->
    <v-dialog v-model="editDialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Edit Task</span>
        </v-card-title>
        <v-card-text>
          <v-text-field
            label="Task Title"
            v-model="taskTitle"
            required
          ></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-btn color="blue darken-1" text @click="closeEditDialog"
            >Cancel</v-btn
          >
          <v-btn color="blue darken-1" text @click="updateTask">Update</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>  

<script>
import Swal from "sweetalert2";
export default {
  data() {
    return {
      tasks: [],
      newTask: {
        title: "",
        date: "",
      },
      filter: "All",
      filterOptions: ["All", "Completed", "Incomplete"],
      addDialog: false,
      editDialog: false,
      viewDialog: false,
    };
  },
  computed: {
    filteredTasks() {
      if (this.filter === "Completed") {
        return this.tasks.filter((task) => task.completed);
      }
      if (this.filter === "Incomplete") {
        return this.tasks.filter((task) => !task.completed);
      }
      return this.tasks;
    },
  },
  methods: {
    showAlert(task) {
      Swal.fire({
        title: "Are you sure?",
        icon: "warning",
        confirmButtonText: "Confirm",
        showCancelButton: true,
        cancelButtonColor: "#d33",
      }).then((result) => {
        if (result.isConfirmed) {
          this.deleteTask(task);
          Swal.fire("Deleted!", "", "success");
        }
      });
    },
    openAddDialog() {
      this.addDialog = true;
      this.newTask.title = "";
      this.newTask.date = "";
    },
    closeAddDialog() {
      this.addDialog = false;
    },
    saveTask() {
      if (this.newTask.title) {
        const newTaskId = this.tasks.length + 1;
        const timestamp = this.formatTimestamp(new Date());
        this.tasks.push({
          id: newTaskId,
          title: this.newTask.title,
          completed: false,
          timestamp: timestamp,
        });
        Swal.fire("Saved!", "", "success");
        this.closeAddDialog();
      } else {
        alert("Please fill out all fields.");
      }
    },
    formatTimestamp(date) {
      const options = {
        hour: "2-digit",
        minute: "2-digit",
        hour12: true,
        day: "2-digit",
        month: "2-digit",
        year: "numeric",
      };
      const formattedDate = date.toLocaleString("en-US", options);
      const [datePart, time] = formattedDate.split(", ");
      return `${time}, ${datePart}`;
    },

    openEditDialog(task) {
      this.currentTaskId = task.id;
      this.taskTitle = task.title;
      this.editDialog = true;
    },
    closeEditDialog() {
      this.editDialog = false;
    },

    updateTask() {
      if (this.taskTitle && this.currentTaskId !== null) {
        const taskIndex = this.tasks.findIndex(
          (task) => task.id === this.currentTaskId
        );
        if (taskIndex !== -1) {
          this.tasks[taskIndex].title = this.taskTitle;
          Swal.fire("Updated!", "", "success");
          this.closeEditDialog();
        }
      }
    },
    openViewDialog(task) {
      this.currentTask = task;
      this.viewDialog = true;
    },
    closeViewDialog() {
      this.viewDialog = false;
      this.currentTask = null;
    },
    deleteTask(task) {
      this.tasks = this.tasks.filter((t) => t.id !== task.id);
    },
    editTask(task) {
      this.openEditDialog(task);
    },
    viewTask(task) {
      this.openViewDialog(task);
    },
  },
};
</script>  

<style scoped>
.container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  background-color: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 20px;
  color: gray;
}

.add-task {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.task-input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ced4da;
  border-radius: 4px;
}

.add-button {
  padding: 10px 15px;
  border: none;
  background-color: #6666ff;
  color: white;
  border-radius: 4px;
  cursor: pointer;
}

.filter-select {
  margin-left: 10px;
}

.task-list {
  border-top: 1px solid #dee2e6;
}

/* .task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #dee2e6;
} */

.task-checkbox {
  margin-right: 10px;
}

.task-title {
  flex: 1;
  margin-top: 12px;
  margin-bottom: 0;
  padding-bottom: 0;
}

.task-date {
  color: #6c757d;
  font-size: 0.85rem;
  margin-left: 10px;
  margin-top: 0;
}

.edit-button,
.delete-button {
  border: none;
  background: none;
  cursor: pointer;
}

.completed {
  text-decoration: line-through;
  color: #6c757d;
}

.headline {
  font-family: "Roboto", sans-serif; /* Use a clean font */
  font-weight: bold;
  color: #333333; /* Darker text color */
}
</style>