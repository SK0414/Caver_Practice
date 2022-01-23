<template>
  <main class="AppWrapper">
    <Login />
  </main>
</template>

<script>
import Login from './components/Login.vue';

export default {
  components: {
    Login,
  },
};
</script>

<style>
.AppWrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40vw;
}
p {
  margin: 0;
}
</style>
