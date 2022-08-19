<script lang="ts">
import { defineComponent } from 'vue';

export default {
  props: {
    heading: String
  }
}
</script>

<template>
  <nav class="navbar navbar-inverse">
    <div class="container-fluid">
      <div class="navbar-header" id="nav">
        <a class="navbar-brand text-wrap" href="#">
          <h2>{{ heading }}</h2>
        </a>
      </div>
    </div>
  </nav>
</template>


<style scoped>
h2 {
  color: #d6d2de;
  font-weight: bold;
}
</style>
