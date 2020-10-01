<template>
  <div class="ui-stepper">
    <stepper-part
      :titles="[
        'Title 1',
        'Title 2',
        'Title 3',
      ]"
    >
      <template #0="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            @click="next"
            look="primary"
            fill="solid">
            Continue
          </ui-button>
        </div>
      </template>
      <template #1="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            @click="next"
            look="primary"
            fill="solid">
            Continue
          </ui-button>
        </div>
      </template>
      <template #2="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            look="primary"
            fill="solid">
            Submit
          </ui-button>
        </div>
      </template>
    </stepper-part>
  </div>
</template>

<script>
import StepperPart from './StepperPart'
import UiButton from '../UiButton/index'

export default {
  components: {
    StepperPart,
    UiButton,
  },
  props: {
  },
  data () {
    return {
      currentStep: 0,
    }
  },
}
</script>

<style lang="scss">
.ui-stepper {
  display: grid;
  gap: 3rem;

  .container-title {
    margin-top: 0;
  }
}
</style>
