<template>
  <div class="app">
    <header class="header">
      <span class="title">Photo Editor</span>
      <Navbar
        :data="data"
        @change="change"
      />
    </header>
    <main class="main">
      <Editor
        v-if="data.loaded"
        ref="editor"
        :data="data"
      />
      <Loader
        v-else
        ref="loader"
        :data="data"
      />
    </main>
  </div>
</template>
<script>
import VueCropper from 'vue-cropperjs';
//import 'cropperjs/dist/cropper.css';

import './styles/index.css';
import { ref } from 'vue';
import Navbar from './components/Navbar.vue';
import Editor from './components/Editor.vue';
import Loader from './components/Loader.vue';

export default {
  components: {
    Navbar,
    Editor,
    Loader,
  },
  setup() {
    const data = ref({
      cropped: false,
      cropping: false,
      loaded: false,
      name: '',
      previousUrl: '',
      type: '',
      url: '',
    });

    const editor = ref(null);
    const loader = ref(null);

    const change = (action) => {
      switch (action) {
        case 'crop':
          editor.value.crop();
          break;
        case 'clear':
          editor.value.clear();
          break;
        case 'restore':
          editor.value.restore();
          break;
        case 'remove':
          editor.value.reset();
          break;
        default:
          break;
      }
    };

    return {
      data,
      editor,
      loader,
      change,
    };
  },
};
</script>
<style scoped>
.app {
  bottom: 0;
  left: 0;
  position: absolute;
  top: 0;
  right: 0;
}

.header {
  background-color: #666;
  height: 3rem;
  overflow: hidden;
  padding-left: 1rem;
  padding-right: 1rem;
  position: relative;
  z-index: 1;
}

@media (min-width: 768px) {
  .header {
    padding-left: 1.5rem;
    padding-right: 1.5rem;
  }
}

.title {
  color: #fff;
  display: block;
  float: left;
  font-size: 1.125rem;
  line-height: 3rem;
}

.main {
  background-color: #333;
  bottom: 0;
  left: 0;
  position: absolute;
  right: 0;
  top: 3rem;
}
</style>
