<template>
  <div
    class="loader"
    @change="change"
    @dragover="dragover"
    @drop="drop"
  >
    <p>
      Paste or drop image here or
      <label class="browse">browse...
        <input
          id="file"
          class="sr-only"
          type="file"
          accept="image/*"
          @change="change"
        >
      </label>
    </p>
  </div>
</template>
<script>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const URL = window.URL || window.webkitURL;
const REGEXP_MIME_TYPE_IMAGES = /^image\/\w+$/;
const REGEXP_URLS = /^(?:https?|data):/;

export default {
  name: 'Loader',

  props: {
    data: {
      type: Object,
      default: () => ({}),
    },
  },

  setup(props) {
    const loader = ref(null);

    const read = (file, event) => {
      return new Promise((resolve, reject) => {
        if (!file) {
          resolve();
          return;
        }

        if (REGEXP_MIME_TYPE_IMAGES.test(file.type)) {
          if (URL) {
            resolve({
              loaded: true,
              name: file.name,
              type: file.type,
              url: URL.createObjectURL(file),
            });
          } else {
            reject(new Error('Your browser is not supported.'));
          }
        } else {
          reject(new Error(`Please ${event ? event.type : 'choose'} an image file.`));
        }
      });
    };

    const change = (event) => {
      const { files } = event.target;

      if (files && files.length > 0) {
        read(files[0], event).then((data) => {
          event.target.value = '';
          update(data);
        }).catch((e) => {
          event.target.value = '';
          alert(e);
        });
      }
    };

    const dragover = (event) => {
      event.preventDefault();
    };

    const drop = (event) => {
      const { files } = event.dataTransfer;

      event.preventDefault();

      if (files && files.length > 0) {
        read(files[0], event)
          .then((data) => {
            update(data);
          })
          .catch(alert);
      }
    };

    const paste = (event) => {
      const { items } = event.clipboardData || window.clipboardData;

      event.preventDefault();

      if (items && items.length > 0) {
        new Promise((resolve, reject) => {
          const item = Array.from(items).pop();
          const error = new Error('Please paste an image file or URL.');

          if (item.kind === 'file') {
            resolve(item.getAsFile());
          } else if (item.kind === 'string') {
            item.getAsString((url) => {
              if (REGEXP_URLS.test(url)) {
                const xhr = new XMLHttpRequest();
                const alert = () => {
                  reject(error);
                };

                xhr.onabort = alert;
                xhr.onerror = alert;
                xhr.ontimeout = alert;

                xhr.onprogress = () => {
                  if (!REGEXP_MIME_TYPE_IMAGES.test(xhr.getResponseHeader('content-type'))) {
                    xhr.abort();
                  }
                };

                xhr.onload = () => {
                  resolve(xhr.response);
                };

                xhr.open('GET', url, true);
                xhr.responseType = 'blob';
                xhr.send();
              } else {
                reject(error);
              }
            });
          } else {
            reject(error);
          }
        })
          .then((blob) => read(blob, event).then((data) => {
            update(data);
          }))
          .catch(alert);
      }
    };

    const alert = (e) => {
      window.alert(e && e.message ? e.message : e);
    };

    const update = (data) => {
      Object.assign(props.data, data);
    };

    onMounted(() => {
      document.addEventListener('paste', paste);
    });

    onBeforeUnmount(() => {
      document.removeEventListener('paste', paste);
    });

    return {
      loader,
      change,
      dragover,
      drop,
      paste,
    };
  },
};
</script>
<style scoped>
.loader {
  display: table;
  height: 100%;
  overflow: hidden;
  width: 100%;

  & > p {
    color: #999;
    display: table-cell;
    text-align: center;
    vertical-align: middle;
  }
}

.browse {
  color: #0074d9;
  cursor: pointer;
  margin-left: .25rem;

  &:hover {
    color: #08f;
    text-decoration: underline;
  }
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0,0,0,0);
  border: 0;
}
</style>

