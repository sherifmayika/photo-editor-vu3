<template>
  <div class="editor">
    <div
      class="canvas"
      @dblclick="dblclick"
    >
      <img
        ref="image"
        :alt="data.name"
        :src="data.url"
        @loadstart="start"
        @load="start"
      >
    </div>
    <div
      v-if="cropper"
      class="toolbar"
      @click="click"
    >
      <button
        class="toolbar__button"
        data-action="move"
        title="Move (M)"
      >
        <span class="fa fa-arrows" />
      </button>
      <button
        class="toolbar__button"
        data-action="crop"
        title="Crop (C)"
      >
        <span class="fa fa-crop" />
      </button>
      <button
        class="toolbar__button"
        data-action="zoom-in"
        title="Zoom In (I)"
      >
        <span class="fa fa-search-plus" />
      </button>
      <button
        class="toolbar__button"
        data-action="zoom-out"
        title="Zoom Out (O)"
      >
        <span class="fa fa-search-minus" />
      </button>
      <button
        class="toolbar__button"
        data-action="rotate-left"
        title="Rotate Left (L)"
      >
        <span class="fa fa-rotate-left" />
      </button>
      <button
        class="toolbar__button"
        data-action="rotate-right"
        title="Rotate Right (R)"
      >
        <span class="fa fa-rotate-right" />
      </button>
      <button
        class="toolbar__button"
        data-action="flip-horizontal"
        title="Flip Horizontal (H)"
      >
        <span class="fa fa-arrows-h" />
      </button>
      <button
        class="toolbar__button"
        data-action="flip-vertical"
        title="Flip Vertical (V)"
      >
        <span class="fa fa-arrows-v" />
      </button>
    </div>
  </div>
</template>
<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import Cropper from 'cropperjs';

export default {
  name: 'Editor',
  props: {
    data: {
      type: Object,
      default: () => ({}),
    },
  },
  setup(props) {
    const canvasData = ref(null);
    const cropBoxData = ref(null);
    const croppedData = ref(null);
    const cropper = ref(null);
    const image = ref(null);

    const update = (newData) => {
      Object.assign(props.data, newData);
    };

    const start = () => {
      if (props.data.cropped || cropper.value) return;

      cropper.value = new Cropper(image.value, {
        autoCrop: false,
        dragMode: 'move',
        background: false,
        ready() {
          if (croppedData.value) {
            cropper.value
              .crop()
              .setData(croppedData.value)
              .setCanvasData(canvasData.value)
              .setCropBoxData(cropBoxData.value);

            croppedData.value = null;
            canvasData.value = null;
            cropBoxData.value = null;
          }
        },
        crop({ detail }) {
          if (detail.width > 0 && detail.height > 0 && !props.data.cropping) {
            update({ cropping: true });
          }
        },
      });
    };

    const stop = () => {
      if (cropper.value) {
        cropper.value.destroy();
        cropper.value = null;
      }
    };

    const crop = () => {
      if (props.data.cropping) {
        croppedData.value = cropper.value.getData();
        canvasData.value = cropper.value.getCanvasData();
        cropBoxData.value = cropper.value.getCropBoxData();
        update({
          cropped: true,
          cropping: false,
          previousUrl: props.data.url,
          url: cropper.value.getCroppedCanvas(props.data.type === 'image/png' ? {} : {
            fillColor: '#fff',
          }).toDataURL(props.data.type),
        });
        stop();
      }
    };

    const clear = () => {
      if (props.data.cropping) {
        cropper.value.clear();
        update({ cropping: false });
      }
    };

    const restore = () => {
      if (props.data.cropped) {
        update({
          cropped: false,
          previousUrl: '',
          url: props.data.previousUrl,
        });
      }
    };

    const reset = () => {
      stop();
      update({
        cropped: false,
        cropping: false,
        loaded: false,
        name: '',
        previousUrl: '',
        type: '',
        url: '',
      });
    };

    const click = ({ target }) => {
      const action = target.getAttribute('data-action') || target.parentElement.getAttribute('data-action');
      switch (action) {
        case 'move':
        case 'crop':
          cropper.value.setDragMode(action);
          break;
        case 'zoom-in':
          cropper.value.zoom(0.1);
          break;
        case 'zoom-out':
          cropper.value.zoom(-0.1);
          break;
        case 'rotate-left':
          cropper.value.rotate(-90);
          break;
        case 'rotate-right':
          cropper.value.rotate(90);
          break;
        case 'flip-horizontal':
          cropper.value.scaleX(-cropper.value.getData().scaleX || -1);
          break;
        case 'flip-vertical':
          cropper.value.scaleY(-cropper.value.getData().scaleY || -1);
          break;
        default:
          break;
      }
    };

    const keydown = (e) => {
      switch (e.key) {
        case 'z':
          if (e.ctrlKey) {
            e.preventDefault();
            restore();
          }
          break;
        case 'Delete':
          reset();
          break;
        default:
          break;
      }

      if (!cropper.value) return;

      switch (e.key) {
        case 'Enter':
          crop();
          break;
        case 'Escape':
          clear();
          break;
        case 'ArrowLeft':
          e.preventDefault();
          cropper.value.move(-1, 0);
          break;
        case 'ArrowUp':
          e.preventDefault();
          cropper.value.move(0, -1);
          break;
        case 'ArrowRight':
          e.preventDefault();
          cropper.value.move(1, 0);
          break;
        case 'ArrowDown':
          e.preventDefault();
          cropper.value.move(0, 1);
          break;
        case 'c':
          cropper.value.setDragMode('crop');
          break;
        case 'm':
          cropper.value.setDragMode('move');
          break;
        case 'i':
          cropper.value.zoom(0.1);
          break;
        case 'o':
          cropper.value.zoom(-0.1);
          break;
        case 'l':
          cropper.value.rotate(-90);
          break;
        case 'r':
          cropper.value.rotate(90);
          break;
        case 'h':
          cropper.value.scaleX(-cropper.value.getData().scaleX || -1);
          break;
        case 'v':
          cropper.value.scaleY(-cropper.value.getData().scaleY || -1);
          break;
        default:
          break;
      }
    };

    const dblclick = (e) => {
      if (e.target.className.includes('cropper-face')) {
        e.preventDefault();
        e.stopPropagation();
        crop();
      }
    };

    onMounted(() => {
      window.addEventListener('keydown', keydown);
    });

    onBeforeUnmount(() => {
      window.removeEventListener('keydown', keydown);
      stop();
    });

    return {
      image,
      click,
      dblclick,
      start,
      stop,
      crop,
      clear,
      restore,
      reset,
      update,
      cropper,
      canvasData,
      cropBoxData,
      croppedData,
    };
  },
};
</script>
<style scoped>
.editor {
  height: 100%;
}

.canvas {
  align-items: center;
  display: flex;
  height: 100%;
  justify-content: center;

  & > img {
    max-height: 100%;
    max-width: 100%;
  }
}

.toolbar {
  background-color: rgba(0, 0, 0, .5);
  bottom: 1rem;
  color: #fff;
  height: 2rem;
  left: 50%;
  margin-left: -8rem;
  position: absolute;
  width: 16rem;
  z-index: 2015;
}

.toolbar__button {
  background-color: transparent;
  border-width: 0;
  color: #fff;
  cursor: pointer;
  display: block;
  float: left;
  font-size: .875rem;
  height: 2rem;
  text-align: center;
  width: 2rem;

  &:focus {
    outline: none;
  }

  &:hover {
    background-color: #0074d9;
    color: #fff;
  }
}
</style>
