<template>
  <div class="ui-photo">
    <div class="ui-photo-clipper">
      <ui-photo-field
        v-if="!isEditorOpened"
        name="ui-photo-clipper-img"
        :value="originImg"
        :note="note"
        :document-type="documentType"
        :min-width="minWidth"
        :min-height="minHeight"
        :disabled="disabled"
        :file-extensions="IMAGE_FILE_EXTENSIONS"
        :max-size="maxSize"
        :error-message="errorMessage"
        @input="tryCropImg"
      />

      <template v-if="isEditorOpened">
        <div
          class="ui-photo-clipper__label"
          :class="{
            'ui-photo-clipper__label--disabled': $attrs.disabled,
            'ui-photo-clipper__label--error': errorMessage
          }"
        >
          {{ labelMain }}
        </div>

        <div class="ui-photo-clipper__editor">
          <clipper-basic
            class="ui-photo-clipper__clipper"
            :src="imgURL"
            ref="clipper"
            bg-color="transparent"
            :ratio="ratio"
          >
            <div
              class="ui-photo-clipper__no-image"
              slot="placeholder">
              <!-- eslint-disable max-len -->
              <i class="mdi mdi-face-recognition ui-photo-clipper__icon ui-photo-clipper__reset-icon" />
              <!-- eslint-enable max-len -->
            </div>
          </clipper-basic>

          <div class="ui-photo-clipper__actions">
            <button
              class="app__button-flat"
              type="button"
              v-if="!$attrs.disabled"
              :title="'clipper-field.save-lbl'"
              :disabled="disabled"
              @click="cropImg"
            >
              {{ labelSave }}
            </button>
            <button
              class="app__button-flat ui-photo-clipper__reset-btn"
              type="button"
              v-if="!$attrs.disabled"
              :title="'clipper-field.cancel-lbl'"
              :disabled="disabled"
              @click="reset"
            >
              {{ labelCancel }}
            </button>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import PhotoField from './PhotoField'
import { clipperBasic } from 'vuejs-clipper'
import Vue from 'vue'
import VueRx from 'vue-rx'
import { Document } from '@tokend/js-sdk'

Vue.use(VueRx)

const MAX_FILE_MEGABYTES = 32
const IMAGE_FILE_EXTENSIONS = ['jpg', 'png', 'jpeg']

export default {
  components: {
    'ui-photo-field': PhotoField,
    clipperBasic,
  },
  props: {
    value: { type: Document, default: null },
    labelMain: { type: String, default: 'Logo (Optional)' },
    labelSave: { type: String, default: 'Save' },
    labelCancel: { type: String, default: 'Cancel' },
    documentType: { type: String, default: 'default' },
    maxSize: { type: Number, default: MAX_FILE_MEGABYTES },
    note: { type: String, default: 'image (Max 32mb)' },
    errorMessage: { type: String, default: undefined },
    minWidth: { type: Number, default: 0 },
    minHeight: { type: Number, default: 0 },
    ratio: { type: Number, default: 1 },
    disabled: { type: Boolean, default: false },
  },

  data () {
    return {
      IMAGE_FILE_EXTENSIONS,
      originImg: null,
      imgURL: '',
      resultURL: '',
      isEditorOpened: false,
    }
  },
  created () {
    this.originImg = this.value
  },
  methods: {
    tryCropImg (value) {
      if (value) {
        this.originImg = value
        if (this.imgURL) URL.revokeObjectURL(this.imgURL)
        this.imgURL = window.URL.createObjectURL(value.file)
        this.isEditorOpened = true
      } else {
        this.reset()
      }
    },
    async cropImg () {
      const canvas = this.$refs.clipper.clip()// call component's clip method
      this.resultURL = canvas.toDataURL(this.originImg.mimeType, 1)

      const blob = this.dataURItoBlob(this.resultURL)
      this.originImg = new Document({
        mimeType: blob.type,
        name: 'new__' + this.originImg.name,
        type: this.documentType,
        file: blob,
      })
      this.isEditorOpened = false
      this.$emit('input', this.originImg)
    },
    reset () {
      if (this.imgURL) URL.revokeObjectURL(this.imgURL)
      this.imgURL = ''
      this.originImg = null
      this.isEditorOpened = false
      this.$emit('input', this.originImg)
    },
    dataURItoBlob (dataURI) {
      // convert base64 to raw binary data held in a string
      // doesn't handle URLEncoded DataURIs - see SO answer
      // #6850276 for code that does this
      let byteString = atob(dataURI.split(',')[1])
      let mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0]
      let ab = new ArrayBuffer(byteString.length)
      let ia = new Uint8Array(ab)

      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i)
      }
      return new Blob([ab], { type: mimeString })
    },
  },
}
</script>

<style lang="scss" scoped>
@import '../../styles/all';

$max-width: 40rem;
.ui-photo-clipper {
  --ui-display: flex;
  --ui-justify-content: center;
  --ui-text-alight: center;
  --ui-align-items: center;
  --ui-font-weight: 700;

  --ui-editor-border: #c4c8cb;
  --ui-editor-backgound-color: #f5f6f9;
  --ui-label-color: #000000;
  --ui-app-button-flat-hover-background-color: #d3daf7;

  width: 100%;
  &__editor {
    display: var(--ui-display);
    flex-direction: column;
    align-items: var(--ui-align-items);
    border: 0.2rem dashed var(--ui-border);
    background-color: var(--ui-editor-backgound-color);
    border-radius: 0.4rem;
    transition: 0.2s;
    text-align: var(--ui-text-alight);
    position: relative;
    padding: 3%;
  }

  &__label {
    font-size: 1.2rem;
    color: var(--ui-label-color);
    margin-bottom: 0.6rem;
  }

  &__label--error {
    color: var(--ui-col-error);
  }

  &__label--disabled {
    filter: grayscale(100%);
  }

  &__clipper {
    max-width: $max-width;
  }

  &__reset-btn{
    color: var(--ui-col-secondary);
  }

  &__icon {
    font-size: 2.4rem;
    display: var(--ui-display);

    &:before {
      font-weight: var(--ui-font-weight);
      vertical-align: middle;
    }
  }

  &__actions {
    display: var(--ui-display);
    justify-content: space-between;
    margin-top: 2rem;
    width: 100%;
    max-width: $max-width;

    .app__button-flat {
      color: var(--ui-col-primary);
      transition: 0.3s ease-out;
      display: var(--ui-display);
      justify-content: var(--ui-justify-content);
      align-items: var(--ui-align-items);
      border: none;
      padding: 0.8rem 1.6rem;
      font-size: 1.4rem;
      font-weight: var(--ui-font-weight);
      text-align: var(--ui-text-alight);
      min-height: 3.6rem;
      background-color: transparent;
      &[disabled] {
        color: var(--ui-col-secondary);
      }

      &:not([disabled]):hover {
        background-color: var(--ui-app-button-flat-hover-background-color);
      }

      &:focus {
        outline: none;
      }
    }
  }

  &__no-image {
    display: var(--ui-display);
    justify-content: var(--ui-justify-content);
    padding: 2rem;
  }
}
</style>
