<template>
  <TimeTable />
  <BootstrapText />
</template>

<script lang="ts">
import { defineComponent } from "vue";
import TimeTable from "./components/TimeTable.vue";

import BootstrapTest from "./components/BootstrapTest.vue";

export default defineComponent({
  name: "App",
  components: {
    TimeTable,
  },
});
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
</style>
