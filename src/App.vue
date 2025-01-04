<template>
  <div id="app">
    <TaskManager />
  </div>
</template>  

<script>
import TaskManager from "./components/TaskManagement.vue";

export default {
  name: "App",
  components: {
    TaskManager,
  },
};
</script>