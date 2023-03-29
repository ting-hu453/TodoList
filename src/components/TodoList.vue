<template>
  <div class="container">
    <h2 class="text-center mt-5 todolist">My Todo List</h2>

    <div class="d-flex">
      <input
        type="text"
        v-model="inputTask"
        placeholder="Enter task"
        class="form-control" />
      <button @click="handleSubmit()" class="btn btn-primary btn-submit">
        Submit
      </button>
    </div>

    <table class="table table-bordered mt-5">
      <thead>
        <tr>
          <th scope="col" class="text-center">Task</th>
          <th scope="col" class="text-center">Status</th>
          <th scope="col" class="text-center">Edit</th>
          <th scope="col" class="text-center">Delete</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in tasks" :key="task.id">
          <td scope="col" class="text-center">
            <span :class="{ completed: task.status === 2 }">{{
              task.title
            }}</span>
          </td>
          <td scope="col" class="text-center" style="width: 120px">
            <span
              class="status"
              @click="handleStatus(task.id, index, task.status)"
              >{{ availableStatuses[task.status] }}</span
            >
          </td>
          <td scope="col">
            <div class="text-center" @click="hanldeEdit(task.id)">
              <span class="fa fa-pen"></span>
            </div>
          </td>
          <td scope="col">
            <div class="text-center" @click="handleDelete(task.id, index)">
              <span class="fa fa-trash"></span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, reactive } from "vue";
const inputTask = ref("");
const edit = ref(null);
const availableStatuses = reactive({
  0: "To-do",
  1: "In-Progress",
  2: "Completed",
});
const tasks = ref([]);

function fetchData() {
  axios
    .get("http://localhost:3000/tasks")
    .then((response) => (tasks.value = response.data));
}

function postData(data) {
  axios
    .post("http://localhost:3000/tasks", data)
    .then((response) => {
      console.log(response);
    })
    .catch((error) => {
      console.log(error);
    });
}

function putData(id, data) {
  axios
    .put(`http://localhost:3000/tasks/${id}`, data)
    .then((response) => {
      console.log(response);
    })
    .catch((error) => {
      console.log(error);
    });
}

function deleteData(id) {
  axios
    .delete(`http://localhost:3000/tasks/${id}`)
    .then((response) => {
      console.log(response);
    })
    .catch((error) => {
      console.log(error);
    });
}

function handleSubmit() {
  if (inputTask.value.length === 0) return;
  if (edit.value === null) {
    tasks.value.push({ title: inputTask.value, status: 0 });
    postData({ title: inputTask.value, status: 0 });
    fetchData();
  } else {
    let currentIndex = tasks.value.findIndex(
      (element) => element.id === edit.value
    );
    tasks.value[currentIndex].title = inputTask.value;
    putData(edit.value, {
      title: inputTask.value,
      status: tasks.value[currentIndex].status,
    });
    edit.value = null;
  }
  inputTask.value = "";
}

function handleDelete(id, index) {
  tasks.value.splice(index, 1);
  deleteData(id);
}

function handleStatus(id, index, taskStatus) {
  const updatedStatus = (taskStatus + 1) % 3;
  tasks.value[index].status = updatedStatus;
  putData(id, {
    title: tasks.value[index].title,
    status: updatedStatus,
  });
}

function hanldeEdit(id) {
  tasks.value.find((item) => {
    if (item.id === id) {
      inputTask.value = item.title;
      edit.value = id;
    }
  });
}

fetchData();
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.status {
  cursor: pointer;
}
.completed {
  text-decoration: line-through;
}
</style>
