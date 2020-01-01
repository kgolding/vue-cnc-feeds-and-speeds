<template>
  <div class="container">
    <h1>CNC Speeds and feeds</h1>
    <cnc-form></cnc-form>
  </div>
</template>

<script>
import CncForm from "./components/CncForm.vue"

// import 'bootstrap';
import 'bootstrap/dist/css/bootstrap.min.css';

export default {
  name: "app",
  components: {
    CncForm
  }
};
</script>
