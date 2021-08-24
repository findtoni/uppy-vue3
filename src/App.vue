<template>
  <section class="bg-gradient h-full w-full">
    <div class="flex flex-col items-center justify-center text-center h-screen">
      <h1 class="text-2xl font-semibold text-center text-white pb-6">Custom Uppy Upload Component in Vue 3</h1>
      <div class="bg-white rounded shadow p-4">
        <FileUpload />
      </div>
    </div>
  </section>
</template>

<script>
import FileUpload from './components/FileUpload.vue';

export default {
  components: {
    FileUpload
  }
}
</script>

<style scoped>
.bg-gradient {
  background: #42275a;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to right, #734b6d, #42275a);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to right, #734b6d, #42275a); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
}
</style>