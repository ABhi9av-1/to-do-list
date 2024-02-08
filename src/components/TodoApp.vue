<template>
  <div class="container" style="max-width: 600px">
    <!-- Heading -->
    <h2 class="text-center mt-5">My Vue Todo App</h2>

    <!-- Input -->
    <div class="d-flex mt-5">
      <input
        type="text"
        v-model="task"
        placeholder="Enter task"
        class="w-100 form-control"
      />
      <button class="btn btn-warning rounded-0" @click="submitTask">
        SUBMIT
      </button>
    </div>

    <!-- Task table -->
    <table class="table table-bordered mt-5">
      <thead>
        <tr>
          <th scope="col">Task</th>
          <th scope="col">Created At</th>
          <th scope="col">Completion Date</th>
          <th scope="col" style="width: 120px">Status</th>
          <th scope="col" class="text-center">#</th>
          <th scope="col" class="text-center">#</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in tasks" :key="task.id">
          <td>
            <span :class="{ 'line-through': task.status === 'finished' }">
              {{ task.name }}
            </span>
          </td>
          <td>{{ formatDate(task.createdAt) }}</td>
          <td>
            {{ task.status === 'finished' ? formatDate(task.completionDate) : 'N/A' }}
          </td>
          <td>
            <span
              class="pointer noselect"
              @click="changeStatus(index)"
              :class="{
                'text-danger': task.status === 'to-do',
                'text-success': task.status === 'finished',
                'text-warning': task.status === 'in-progress',
              }"
            >
              {{ capitalizeFirstChar(task.status) }}
            </span>
          </td>
          <td class="text-center">
            <div @click="deleteTask(index)">
              <span class="fa fa-trash pointer"></span>
            </div>
          </td>
          <td class="text-center">
            <div @click="editTask(index)">
              <p class="fa fa-pen pointer"></p>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  props: {
    msg: String,
  },

  data() {
    return {
      task: "",
      editedTask: null,
      statuses: ["to-do", "in-progress", "finished"],
      tasks: [],
    };
  },

  created() {
    this.fetchTasks();
  },

  methods: {
    async fetchTasks() {
      try {
        const response = await fetch("http://localhost:3000/tasks");
        this.tasks = await response.json();
      } catch (error) {
        console.error("Error fetching tasks:", error);
      }
    },

    async submitTask() {
      if (this.task.length === 0) return;

      if (this.editedTask != null) {
        const editedTask = this.tasks[this.editedTask];
        editedTask.name = this.task;
        try {
          await fetch(`http://localhost:3000/tasks/${editedTask.id}`, {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(editedTask),
          });
          this.editedTask = null;
        } catch (error) {
          console.error("Error updating task:", error);
        }
      } else {
        const newTask = {
          name: this.task,
          status: "to-do",
          createdAt: new Date().toISOString(),
          completionDate: null
        };
        try {
          const response = await fetch("http://localhost:3000/tasks", {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(newTask),
          });
          const data = await response.json();
          this.tasks.push(data);
        } catch (error) {
          console.error("Error adding task:", error);
        }
      }

      this.task = "";
    },

    async changeStatus(index) {
      const task = this.tasks[index];
      const newIndex = (this.statuses.indexOf(task.status) + 1) % this.statuses.length;
      task.status = this.statuses[newIndex];
      if (task.status === 'finished') {
        task.completionDate = new Date().toISOString();
      } else {
        task.completionDate = null;
      }

      try {
        await fetch(`http://localhost:3000/tasks/${task.id}`, {
          method: "PUT",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(task),
        });
      } catch (error) {
        console.error("Error updating task status:", error);
      }
    },

    async deleteTask(index) {
      const taskId = this.tasks[index].id;
      this.tasks.splice(index, 1);
      try {
        await fetch(`http://localhost:3000/tasks/${taskId}`, {
          method: "DELETE",
        });
      } catch (error) {
        console.error("Error deleting task:", error);
      }
    },

    editTask(index) {
      this.task = this.tasks[index].name;
      this.editedTask = index;
    },

    capitalizeFirstChar(str) {
      return str.charAt(0).toUpperCase() + str.slice(1);
    },

    formatDate(dateString) {
      if (!dateString) return 'N/A';
      const date = new Date(dateString);
      const formattedDate = date.toLocaleDateString('en-US', {
        year: 'numeric',
        month: 'numeric',
        day: 'numeric',
        hour: 'numeric',
        minute: 'numeric',
        second: 'numeric',
        timeZone: 'UTC',
        timeZoneName: 'short'
      });
      return formattedDate;
    },
  },
};
</script>

<style scoped>
.pointer {
  cursor: pointer;
}
.noselect {
  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none; /* Safari */
  -khtml-user-select: none; /* Konqueror HTML */
  -moz-user-select: none; /* Old versions of Firefox */
  -ms-user-select: none; /* Internet Explorer/Edge */
  user-select: none; /* Non-prefixed version, currently
                                  supported by Chrome, Edge, Opera and Firefox */
}
.line-through {
  text-decoration: line-through;
}
</style>