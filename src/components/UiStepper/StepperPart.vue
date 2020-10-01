<template>
  <div class="stepper-part">
    <template v-for="(title, i) in titles">
      <div
        :key="i"
        :class="{
          'stepper-part__step': true,
          'stepper-part__step--active': slotNames[i] === current,
        }"
      >
        <button
          class="stepper-part__button"
          @click="current = slotNames[i]"
          :disabled="$attrs.disabled || slotNames[i] > current">
          <p class="stepper-part__title">
            <span class="stepper-part__circle">
              <template v-if="slotNames[i] < current">
                <i class="stepper-part__completed-icon mdi mdi-check" />
              </template>
              <template v-else>
                {{ i+1 }}
              </template>
            </span>
            {{ title }}
          </p>
        </button>

        <div class="stepper-part__step-content">
          <transition name="content">
            <template v-if="current === slotNames[i]">
              <slot
                :next="next"
                :name="slotNames[i]"
              />
            </template>
          </transition>
        </div>
      </div>
    </template>
  </div>
</template>

<script>

export default {
  props: {
    titles: {
      type: Array,
      required: true,
    },
  },
  data: _ => ({
    current: null,
  }),
  computed: {
    isLastSelected () {
      return this.current === this.titles.length - 1
    },
    slotNames () {
      return Object.keys(this.$scopedSlots).sort((a, b) => a - b)
    },
  },
  mounted () {
    this.current = this.slotNames[0]
  },
  methods: {
    next () {
      const currentIdx = this.slotNames.findIndex(n => n === this.current)
      this.current = this.slotNames[currentIdx + 1]
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../../styles/all";

.stepper-part {
  overflow: hidden;

  --circle-size: 3rem;
  --circle-margin: calc(var(--circle-size) / 3);

  &__step {
    position: relative;

    &-content {
      overflow: hidden;
      margin: 0.5rem calc(var(--circle-size) / 2);
      padding: 1rem calc(var(--circle-size) / 2 + var(--circle-margin));
    }

    &:not(:last-child) &-content {
      border-left: 0.1rem solid black;
    }
  }

  &__title {
    display: flex;
    align-items: center;
  }

  &__button {
    background-color: transparent;
    border: none;

    &:disabled {
      cursor: default;
      filter: grayscale(100%);
      opacity: 0.5;
    }
    &:focus {
      outline: none;
    }
  }

  &__circle {
    display: flex;
    justify-content: center;
    align-items: center;
    background: var(--ui-col-primary);
    color: var(--ui-col-primary-contrast);
    border-radius: 100%;
    font-size: calc(var(--circle-size) / 2);
    margin-right: var(--circle-margin);
    width: var(--circle-size);
    height: var(--circle-size);
    position: relative;
  }
}

.content-enter-active,
.content-leave-active {
  transition: all 0.2s linear;
  max-height: 100vh;
}

.content-enter,
.content-leave-to {
  opacity: 0;
  max-height: 0;
}

</style>
