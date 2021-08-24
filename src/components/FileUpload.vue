<template>
  <div class="UppyForm">
    <StatusBar :uppy="uppy" :props="{hideUploadButton: true, hideAfterFinish: true}" />
    <div class="grid grid-cols-2 md:grid-cols-3 gap-4 my-2">
      <div v-for="(file, index) in thumbnails" :key="index" class="space-y-1.5">
        <img :src="file.source" class="rounded h-28 w-28 object-cover">
        <p class="text-sm truncate">{{ file.name }}</p>
        <div class="flex justify-between items-center">
          <div class="flex flex-col">
            <p v-if="file.size <= 999000" class="text-sm">{{ (file.size/1000).toFixed(1) }} kb</p>
            <p v-else class="text-sm c-text-body">{{ (file.size/10000).toFixed(1) }} MB</p>
          </div>
          <p @click="remove(file.id)" class="text-xs text-red-600 cursor-pointer">Remove</p>
        </div>
      </div>
      <form :action="endpoint">
        <div class="h-28 w-28 bg-white shadow-md border-gray-700 flex flex-col justify-center items-center p-6 rounded">
          <img src="../assets/upload-icon.svg" width="25">
          <input @change="upload" type="file" class="upload" id="upload" multiple="multiple" hidden>
          <label for="upload" class="text-sm font-semibold text-purple mt-2 cursor-pointer">Upload</label>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { defineComponent, onBeforeUnmount, ref } from 'vue';
import Uppy from '@uppy/core';
import XHRUpload from '@uppy/xhr-upload';
import ThumbnailGenerator from '@uppy/thumbnail-generator';
import { StatusBar } from '@uppy/vue';
import '@uppy/core/dist/style.css';
import '@uppy/status-bar/dist/style.css';

export default defineComponent({
  name: 'FileUpload',
  props: {
    name: {
      type: String,
      required: false,
    },
    endpoint: {
      type: String,
      default: 'https://xhr-server.herokuapp.com/upload',
      required: false,
    },
    allowedFileTypes: {
      type: Array,
      default: null,
      required: false,
    }
  },
  components: {
    StatusBar,
  },
  setup(props, { emit }) {
    const thumbnails = ref([]);
    const uppy = new Uppy({
      debug: true,
      autoProceed: true,
      allowMultipleUploads: true,
      restrictions: {
        maxFileSize: 5000000,
        maxNumberOfFiles: 10,
        minNumberOfFiles: 1,
        allowedFileTypes: props.allowedFileTypes,
      }
    });

    uppy.use(ThumbnailGenerator, {
      id: 'ThumbnailGenerator',
      thumbnailWidth: 200,
      thumbnailHeight: 200,
      thumbnailType: 'image/jpeg',
      waitForThumbnailsBeforeUpload: false,
    });
    uppy.use(XHRUpload, {
      endpoint: props.endpoint,
      formData: true,
      fieldName: 'files[]',
      limit: 10,
    });
    uppy.on('upload-success', (file, response) => {
      emit('success', response.uploadURL);
    });
    uppy.on('thumbnail:generated', (file, preview) => {
      thumbnails.value.push({
        id: file.id,
        name: file.name,
        source: preview,
        size: file.size,
      });
    });

    function upload(e) {
      const files = Array.from(e.target.files);
      files.forEach((file) => {
        try {
          uppy.addFile({
            source: props.name,
            name: file.name,
            type: file.type,
            data: file,
          })
        } catch (error) {
          emit('error', error);
        }
      });
    }
    function remove(file) {
      uppy.removeFile(file);
      thumbnails.value = thumbnails.value.filter(item => item.id !== file);
    }

    onBeforeUnmount(() => {
      uppy.close();
    });

    return {
      uppy,
      thumbnails,
      upload,
      remove,
    }
  },
  emits: ['success', 'error'],
})
</script>

<style scoped>
.text-purple {
  color: #734b6d;
}
</style>