<template>
  <div id="app">
    <FileUpload class="file-upload" msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import FileUpload from './components/FileUpload.vue'

export default {
  name: 'app',
  components: {
    FileUpload
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  height: 100vh;

}
.logo {
  height: 40em;
}
.file-upload {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%);
}
</style>
