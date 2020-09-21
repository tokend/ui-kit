<template>
  <div class="ui-photo-field">
    <div
      class="ui-photo-field__label"
      :class="{
        'ui-photo-field__label--disabled': $attrs.disabled,
        'ui-photo-field__label--error': errorMessage
      }"
    >
      {{ labelMain }}
    </div>

    <div
      class="ui-photo-field__content"
      :class="{
        'ui-photo-field__content--disabled': $attrs.disabled,
        'ui-photo-field__content--error': errorMessage,
        'ui-photo-field__content--highlighted': isFileDragged
      }"
    >
      <template v-if="document">
        <button
          v-if="!$attrs.disabled"
          :title="'ui-photo-field.reset-btn-hint'"
          class="button_icon ui-photo-field__reset-btn"
          type="button"
          @click="reset"
        >
          <i class="mdi mdi-close ui-photo-field__reset-icon" />
        </button>

        <div
          v-if="documentUrl && isImageSelected"
          class="ui-photo-field__img-preview-wrp"
        >
          <img
            class="ui-photo-field__img-preview"
            :src="documentUrl">
        </div>

        <div
          v-else
          class="ui-photo-field__icon-preview-wrp">
          <i class="mdi mdi-file ui-photo-field__icon-preview" />
        </div>

        <div
          class="ui-photo-field__selected-file">
          {{ labelFileName }}
        </div>
      </template>

      <div
        class="ui-photo-field__inner">
        <template v-if="!$attrs.disabled">
          <i
            v-if="!document"
            class="mdi ui-photo-field__icon"
            :class="isFileDragged ? 'mdi-file-plus' : 'mdi-upload'"
          />

          <div class="ui-photo-field__text">
            <p
              :class="document
                ? 'ui-photo-field__subtitle'
                : 'ui-photo-field__title'"
            >
              <template v-if="isFileDragged">
                {{ labelDraged }}
              </template>

              <template v-else-if="document">
                {{ labelDraged }}
              </template>

              <template v-else>
                {{ labelTitle }}
              </template>
            </p>

            <div class="ui-photo-field__note">
              {{ note || acceptedExtensions }}
            </div>
          </div>
        </template>

        <template v-else-if="!document">
          <i class="mdi mdi-file-hidden ui-photo-field__icon" />

          <div class="ui-photo-field__text">
            <p class="ui-photo-field__title">
              {{ 'ui-photo-field.no-file-selected-title' }}
            </p>
          </div>
        </template>
      </div>

      <input
        v-bind="$attrs"
        type="file"
        class="ui-photo-field__input"
        :accept="acceptedExtensions"
        title=""
        @change="onChange"
        @dragenter="isFileDragged = true"
        @dragleave="isFileDragged = false"
        @drop="isFileDragged = false"
      >
    </div>

    <div
      class="ui-photo-field__err-mes"
      v-if="errorMessage"
    >
      {{ errorMessage }}
    </div>
  </div>
</template>

<script>
import { FileUtil, FileNotPresentInEventError } from '@/utils/file.util'
import { Document } from '@tokend/js-sdk'

const MAX_FILE_MEGABYTES = 32
const FILE_EXTENSIONS = ['jpg', 'png', 'jpeg', 'pdf']

export default {
  props: {
    value: { type: Document, default: null },
    labelMain: { type: String, default: 'Logo (Optional)' },
    labelTitle: { type: String, default: 'Drag a file here or click to browse' },
    labelFileName: { type: String, default: 'Selected file: ' },
    labelDraged: { type: String, default: 'Drag another file or click to browse' },
    documentType: { type: String, default: 'default' },
    fileExtensions: { type: Array, default: _ => FILE_EXTENSIONS },
    maxSize: { type: Number, default: MAX_FILE_MEGABYTES },
    note: { type: String, default: 'image (Max 32mb)' },
    errorMessage: { type: String, default: undefined },
    minWidth: { type: Number, default: 0 },
    minHeight: { type: Number, default: 0 },
  },

  data: _ => ({
    document: null,
    isFileDragged: false,
    documentUrl: '',
  }),

  computed: {
    maxSizeBytes () {
      return this.maxSize * 1024 * 1024
    },

    isImageSelected () {
      const mimeType = this.document.mimeType
      return mimeType && mimeType.includes('image')
    },

    acceptedExtensions () {
      return this.fileExtensions
        .map(item => `.${item.toUpperCase()}`)
        .join(', ')
    },
  },
  watch: {
    'value': {
      handler (value) {
        this.document = value
        if (value) this.loadDocumentUrl(value)
      },
      immediate: true
    }
  },
  methods: {
    async loadDocumentUrl (document) {
      if (document.key) {
        this.documentUrl = await document.getPrivateUrl()
      } else {
        this.documentUrl = await FileUtil.getDataUrl(document.file)
      }
    },
    async onChange (event) {
      try {
        const file = FileUtil.getFileFromEvent(event)
        if (await this.validateFile(file)) {
          this.documentUrl = await FileUtil.getDataUrl(file)
          this.document = Document({
            mimeType: file.type,
            type: this.documentType,
            name: file.name,
            file: file,
          })
          this.$emit('input', this.document)
        } else {
          this.$emit('error')
        }
      } catch (e) {
        if (e instanceof FileNotPresentInEventError) {
          this.reset()
        } else {
          console.error(e)
          this.$emit('error')
        }
        return false
      }
    },

    reset () {
      this.$el.querySelector('input').value = ''
      this.document = null
      this.documentUrl = ''
      this.$emit('input', this.document)
    },

    async validateFile (file) {
      if (!this.isValidFileType(file)) {
        this.$emit('error')
        return false
      }

      if (!this.isValidFileSize(file)) {
        this.$emit('error')
        return false
      }

      if (!(await this.isValidFileDimensions(file))) {
        this.$emit('error')
        return false
      }
      return true
    },

    async isValidFileDimensions (file) {
      if (!file.type.includes('image')) {
        return true
      }
      try {
        const img = await FileUtil.readImage(file)
        return img.width >= this.minWidth &&
          img.height >= this.minHeight
      } catch (e) {
        return false
      }
    },

    isValidFileType (file) {
      return Boolean(this.fileExtensions
        .find(item => item.toUpperCase() === this.getFileExtension(file))
      )
    },

    getFileExtension (file) {
      return file.name.split('.').pop().toUpperCase()
    },

    isValidFileSize (file) {
      return file.size <= this.maxSizeBytes
    },
  },
}
</script>

<style lang='scss' scoped>
@import '../../styles/all';

$z-reset-btn: 1;
.ui-photo-field {
  --ui-font-size: 1.2rem;
  --ui-width: 100%;
  --ui-top: 0;
  --ui-left: 0;
  --ui-right: 0;
  --ui-text-align: center;
  --ui-font-weight: 700;
  --ui-cursor: pointer;
  --ui-display: flex;
  --ui-position: absolute;

  --ui-label-color: #646464;
  --ui-icon-color: #837fa1;
  --ui-button-icon-hover-background-color: #d3daf7;
  --ui-content-border-color: #000000;
  --ui-title-color: #212121;
  --ui-content-border: #c4c8cb;
  --ui-content-background-color: #f5f6f9;
  --ui-content-highlighted: #212121;
  --ui-content-highlighted: #f5f6ff;
  --ui-selected-file-color: #837fa1;

  &__label {
    color: var(--ui-label-color);
    font-size: var(--ui-font-size);
    line-height: 1;
  }

  &__inner {
    margin: 1rem 0;
    overflow: hidden;
  }

  &__input {
    position: var(--ui-position);
    top: var(--ui-top);
    left: var(--ui-left);
    cursor: var(--ui-cursor);
    opacity: 0;
    height: 100%;
    width: var(--ui-width);
  }

  &__icon {
    color: var(--ui-icon-color);
    font-size: 4.2rem;
  }

  &__text {
    display: var(--ui-display);
    flex-direction: column;
    justify-content: center;
  }

  &__title {
    color: var(--ui-title-color);
    font-size: 1.8rem;
    margin: 0;
  }

  &__note {
    color: var(color-dark-grey-purple);
    margin-top: 0.6rem;
    font-size: var(--ui-font-size);
    line-height: 2.2rem;
  }

  &__label--error {
    color: var(--ui-col-error);
  }

  &__content {
    margin-top: 0.6rem;
    border: 0.2rem dashed var(--ui-content-border);
    background-color: var(--ui-content-background-color);
    border-radius: 0.4rem;
    transition: 0.2s;
    text-align: var(--ui-text-align);
    position: relative;
    min-height: 9.8rem;

    &:not(.file-field__content--disabled):hover {
      border-color: var(--ui-content-border-color);
    }
  }

  &__content--error {
    border-color: var(--ui-col-error);
  }

  &__content--highlighted {
    background-color:  var(--ui-content-highlighted-background-color);
    border-color: var(--ui-content-highlighted-border-color);
  }

  &__content--disabled,
  &__label--disabled {
    filter: grayscale(100%);
  }

  &__img-preview-wrp {
    margin-top: 2rem;
    width: var(--ui-width);
  }

  &__img-preview {
    padding: 0 5rem;
    object-fit: contain;
    height: 15rem;
    margin: auto;
  }

  &__reset-btn {
    position: var(--ui-position);
    z-index: $z-reset-btn;
    top: var(--ui-top);
    right: var(--ui-right);
    min-width: 3rem;
    min-height: 3rem;
    margin: 0.6rem;
    color: var(--ui-col-secondary);
  }

  &__reset-icon {
    font-size: 2.4rem;

    &:before {
      font-weight: var(--ui-font-weight);
      vertical-align: middle;
    }
  }

  &__icon-preview {
    font-size: 8rem;
  }

  &__selected-file {
    color:  var(--ui-selected-file-color);
    font-size: var(--ui-font-size);
    margin-bottom: 2rem;
    word-break: break-all;
    padding: 0 1rem;
  }

  &__subtitle {
    color: var(--ui-selected-text-color);
    font-size: 1.6rem;
  }

  &__err-mes {
    color: var(--ui-col-error);
    margin-top: 0.4rem;
    font-size: var(--ui-font-size);
    line-height: 1.25;
  }
}

.button_icon {
  display: var(--ui-display);
  justify-content: center;
  align-items: center;
  border: none;
  padding: 0.8rem 1.6rem;
  font-size: 1.4rem;
  font-weight: var(--ui-font-weight);
  text-align: var(--ui-text-align);
  min-height: 3.6rem;
  background-color: transparent;
  padding: 0.5rem;
  min-width: 3.6rem;
  border-radius: 50%;
  transition: 0.3s ease-out;
  box-sizing: content-box;
  color: var(--ui-col-primary);

  &:hover {
    background-color: var( --ui-button-icon-hover-background-color);
  }
}

.mdi {
  &:before {
    display: inline-block;
    font: normal normal normal 24px/1 "Material Design Icons";
    font-size: inherit;
    text-rendering: auto;
    line-height: inherit;
    -webkit-font-smoothing: antialiased;
  }
}

.mdi-close {
  font-weight: var(--ui-font-weight);
  vertical-align: middle;
}
</style>
