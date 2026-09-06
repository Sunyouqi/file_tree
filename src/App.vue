<template>
  <h1>You did it!</h1>
  <p>
    Visit <a href="https://vuejs.org/" target="_blank" rel="noopener">vuejs.org</a> to read the
    documentation
  </p>
  <div>
    <vue-finder id="my_vuefinder" :driver="driver" :config="{
      initialPath: 'local://public',
      persist: true,
    }" />

    <file-tree-list />
  </div>
</template>

<script setup lang="ts">
import { RemoteDriver } from 'vuefinder';
import FileTree from './components/FileTree.vue';
import FileTreeList from './components/FileTreeList.vue';
import { Draggable } from "@he-tree/vue";
import "@he-tree/vue/style/default.css";
import { reactive, defineComponent, onMounted } from "vue";
import { ref } from 'vue'



// Update the data when Component A emits it


const driver = new RemoteDriver({
  baseURL: '/api',
  url: {
    list: '/files',
    upload: '/upload',
    delete: '/delete',
    rename: '/rename',
    archive: '/archive',
    unarchive: '/unarchive',
    createFile: '/create-file',
    createFolder: '/create-folder',
    search: '/search',
    preview: '/preview',
    copy: '/copy',
    move: '/move',
    save: '/save',
    download: '/download',
  },
});

onMounted(() => { });
</script>
<style scoped></style>
