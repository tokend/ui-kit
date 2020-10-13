<template>
  <div class="ui-photo">
    <div class="ui-photo-clipper">
      <ui-photo-field
        name="ui-photo-clipper-img"
        :value="originImg"
        :document-type="documentType"
        :min-width="minWidth"
        :min-height="minHeight"
        :disabled="disabled"
        :file-extensions="IMAGE_FILE_EXTENSIONS"
        :max-size="maxSize"
        :error-message="errorMessage"
        :error-disabled="disabled"
        @input="tryCropImg"
        @err-invalid-file-size="$emit('err-invalid-file-size')"
        @err-invalid-file-type="$emit('err-invalid-file-type')"
        @err-invalid-file-dimensions="$emit('err-invalid-file-dimensions')"
      >
        <slot
          name="error"
          slot="error" />
      </ui-photo-field>
      <transition
        name="ui-photo-clipper__modal"
        v-if="isEditorOpened">
        <div class="ui-photo-clipper__modal-mask">
          <div class="ui-photo-clipper__modal-wrapper">
            <div class="ui-photo-clipper__modal-container">
              <div class="ui-photo-clipper__modal-body">
                <div
                  class="ui-photo-clipper__label"
                  :class="{
                    'ui-photo-clipper__label--disabled': $attrs.disabled,
                    'ui-photo-clipper__label--error': errorMessage,
                  }"
                >
                  <slot name="mainLabel" />
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
                      ƒ
                      <!-- eslint-disable max-len -->
                      <i
                        class="mdi mdi-face-recognition ui-photo-clipper__icon ui-photo-clipper__reset-icon"
                      />
                      <!-- eslint-enable max-len -->
                    </div>
                  </clipper-basic>

                  <div class="ui-photo-clipper__actions">
                    <ui-button
                      look="secondary"
                      :title="'clipper-field.cancel-lbl'"
                      fill="none"
                      @click="reset"
                      full-width
                    >
                      <slot name="cancelSavingDragedPhoto" />
                    </ui-button>
                    <ui-button
                      look="primary"
                      :title="'clipper-field.save-lbl'"
                      @click="cropImg"
                      full-width
                    >
                      <slot name="saveDragedPhoto" />
                    </ui-button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import PhotoField from './PhotoField'
import { clipperBasic } from 'vuejs-clipper'
import Vue from 'vue'
import VueRx from 'vue-rx'
import { Document } from '@tokend/js-sdk'

import { UiButton } from '../../index'

Vue.use(VueRx)

const MAX_FILE_MEGABYTES = 32
const IMAGE_FILE_EXTENSIONS = ['jpg', 'png', 'jpeg']

export default {
  components: {
    'ui-photo-field': PhotoField,
    UiButton,
    clipperBasic,
  },
  props: {
    value: {
      type: Document,
      default: null,
    },
    documentType: {
      type: String,
      default: 'default',
    },
    maxSize: {
      type: Number,
      default: MAX_FILE_MEGABYTES,
    },
    errorMessage: {
      type: String,
      default: undefined,
    },
    minWidth: {
      type: Number,
      default: 0,
    },
    minHeight: {
      type: Number,
      default: 0,
    },
    ratio: {
      type: Number,
      default: 1,
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    errorDisabled: {
      type: Boolean,
      default: false,
    },
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
      const canvas = this.$refs.clipper.clip() // call component's clip method
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
      return new Blob([ab], {
        type: mimeString,
      })
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../../styles/all";

$max-width: 40rem;

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active до версии 2.1.8 */
{
  opacity: 0;
}

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
  font-family: var(--ui-font-family);

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
    text-align: var(--ui-text-alight);
    color: var(--ui-label-color);
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

  &__reset-btn {
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
  }

  &__no-image {
    display: var(--ui-display);
    justify-content: var(--ui-justify-content);
    padding: 2rem;
  }

  &__modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: table;
    transition: opacity 0.3s ease;
  }

  &__modal-wrapper {
    display: table-cell;
    vertical-align: middle;
  }

  &__modal-container {
    width: 50rem;
    margin: 0px auto;
    background-color: #fff;
    border-radius: 2px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
    transition: all 0.3s ease;
  }

  &__modal-header h3 {
    margin-top: 0;
    text-align: center;
    text-transform: uppercase;
    color: var(--ui-text-alight);
  }

  &__modal-body {
    margin: 20px 0;
  }

  &__modal-container {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }

  &__modal-enter &__modal-container,
  &__modal-leave-active &__modal-container {
    -webkit-transform: scale(1.1);
    transform: scale(1.1);
  }

  &__modal-enter {
    opacity: 0;
  }

  &__modal-leave-active {
    opacity: 0;
  }
}
</style>
