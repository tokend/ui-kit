<template>
  <div class="ui-photo-field">
    <div class="ui-photo-field__container">
      <div
        class="ui-photo-field__content"
        :class="{
          'ui-photo-field__content--disabled': $attrs.disabled,
          'ui-photo-field__content--highlighted': isFileDragged,
        }"
      >
        <template v-if="document">
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
            v-if="!$attrs.disabled"
            class="ui-photo-field__change-preview-wrp"
          >
            <span class="ui-photo-field__change-preview">Change photo</span>
          </div>
        </template>

        <div class="ui-photo-field__inner">
          <i
            v-if="!document"
            class="mdi ui-photo-field__icon"
            :class="fieldIcon"
          />
          <div
            v-if="!document"
            class="ui-photo-field__label"
          >
            Your photo
          </div>
        </div>

        <input
          v-bind="$attrs"
          type="file"
          class="ui-photo-field__input"
          :accept="acceptedExtensions"
          @click="reset"
          @change="onChange"
          @dragenter="isFileDragged = true"
          @dragleave="isFileDragged = false"
          @drop="isFileDragged = false"
        >
      </div>

      <div
        class="ui-photo-field__err-msg"
        v-if="hasSlot('error')">
        <slot name="error" />
      </div>
    </div>
  </div>
</template>

<script>
import { FileUtil, FileNotPresentInEventError } from '../../utils/file.util'
import { Document } from '@tokend/js-sdk'

const MAX_FILE_MEGABYTES = 32
const FILE_EXTENSIONS = ['jpg', 'png', 'jpeg']

export default {
  props: {
    value: {
      type: Document,
      default: null,
    },
    documentType: {
      type: String,
      default: 'default',
    },
    fileExtensions: {
      type: Array,
      default: (_) => FILE_EXTENSIONS,
    },
    maxSize: {
      type: Number,
      default: MAX_FILE_MEGABYTES,
    },
    minWidth: {
      type: Number,
      default: 0,
    },
    minHeight: {
      type: Number,
      default: 0,
    },
    fieldIcon: {
      type: String,
      default: '',
    }
  },

  data: (_) => ({
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
        .map((item) => `.${item.toUpperCase()}`)
        .join(', ')
    },
  },
  watch: {
    value: {
      handler (value) {
        this.document = value
        if (value) this.loadDocumentUrl(value)
      },
      immediate: true,
    },
  },
  methods: {
    hasSlot (slot) {
      return !!this.$slots[slot]
    },
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
          this.document = new Document({
            mimeType: file.type,
            name: file.name,
            file: file,
          }, this.documentType)
          this.$emit('input', this.document)
        } else {
          this.$emit('error')
        }
      } catch (e) {
        if (e instanceof FileNotPresentInEventError) {
          this.reset()
        } else {
          this.$emit(e)
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
        this.$emit('err-invalid-file-type')
        return false
      }

      if (!this.isValidFileSize(file)) {
        this.$emit('err-invalid-file-size')
        return false
      }

      if (!(await this.isValidFileDimensions(file))) {
        this.$emit('err-invalid-file-dimensions')
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
        return img.width >= this.minWidth && img.height >= this.minHeight
      } catch (e) {
        return false
      }
    },

    isValidFileType (file) {
      return Boolean(
        this.fileExtensions.find(
          (item) => item.toUpperCase() === this.getFileExtension(file)
        )
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

<style lang="scss" scoped>
@import "../../styles/all";

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
    text-align: var(--ui-text-align);
    font-size: var(--ui-font-size);
    line-height: 1;
  }

  &__inner {
    display: var(--ui-display);
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }

  &__input {
    position: var(--ui-position);
    top: var(--ui-top);
    left: var(--ui-left);
    cursor: var(--ui-cursor);
    opacity: 0;
    width: var(--ui-width);
    height: 100%;
    &[disabled]{
      cursor: auto;
    }
  }

  &__icon {
    color: var(--ui-icon-color);
    font-size: 7rem;
  }

  &__container {
    display: var(--ui-display);
    align-items: center;
    flex-direction: column;
  }

  &__content {
    display: var(--ui-display);
    justify-content: center;
    width: 15rem;
    height: 15rem;
    position: relative;
    border-radius: 50%;
    border: 0.2rem solid #c4c8cb;
    background-color: var(--ui-content-background-color);
    transition: 0.2s;
    overflow: hidden;
  }

  &__content:hover &__change-preview-wrp {
    opacity: 1;
  }

  &__content--highlighted {
    background-color: var(--ui-content-highlighted-background-color);
    border-color: var(--ui-content-highlighted-border-color);
  }

  &__content--disabled {
    filter: grayscale(100%);
    opacity: 0.5;
  }

  &__img-preview-wrp {
    display: var(--ui-display);
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: var(--ui-width);
  }

  &__img-preview {
    border-radius: 50%;
    object-fit: contain;
    width: 100%;
    height: 100%;
  }

  &__icon-preview {
    font-size: 8rem;
  }

  &__change-preview-wrp {
    position: absolute;
    display: var(--ui-display);
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 5.5rem;
    bottom: 0;
    color: var(--ui-label-color);
    font-size: var(--ui-font-size);
    border-radius: 0 0 50% 50%;
    background: var(--ui-content-background-color);
    transition: 0.5s;
    opacity: 0;
  }

  &__err-msg {
    display: block;
    color: var(--ui-col-error);
    font-size: var(--ui-font-size);
    margin-top: 0.5rem;
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
