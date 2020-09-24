<template>
  <div
    class="ui-text"
    :for="inputId">
    <span
      class="ui-text__dock ui-text__dock_left"
      v-if="hasSlot('dock-left')">
      <!--
        @slot Optional slot left from the input.
        Will npt be rendered if empty.
      -->
      <slot name="dock-left" />
    </span>
    <span
      class="ui-text__dock ui-text__dock_right"
      v-if="hasSlot('dock-right')"
    >
      <!--
        @slot Optional slot right from the input.
        Will npt be rendered if empty.
      -->
      <slot name="dock-right" />
    </span>
    <input
      class="ui-text__input"
      :id="inputId"
      :look="look"
      :fill="fill"
      :is-required="isRequired"
      :is-disabled="isDisabled"
      :is-error="isError"
      :disabled="isDisabled"
      :placeholder="placeholder"
      :value="value"
      :type="type"
      @input="input"
      v-on="reListeners"
      v-bind="$attrs"
    />

    <span
      class="ui-text__label ui-text__span ui-text__label_required-star "
      :fill="fill"
    >
      {{ placeholder /* for position purposes, see css */ }}
    </span>

    <span
      class="ui-text__label"
      v-if="hasSlot('default') && (!hasSlot('error') || !isError)"
    >
      <!-- @slot Use this slot to place the text field label content -->
      <slot />
    </span>
    <span
      class="ui-text__label ui-text__label_is-error"
      v-if="hasSlot('error') && isError"
    >
      <!-- @slot Content of this slot will be visible if ``:is-error`` true -->
      <slot name="error" />
    </span>
  </div>
</template>

<script>
export default {
  name: 'UiText',
  inheritAttrs: false,
  props: {
    /** Regular value */
    value: { type: [String, Number], default: '' },
    /** Override input[type] property. */
    type: { type: String, default: 'text' },
    /** Regular placeholder */
    placeholder: { type: String, default: '' },
    /**
     * Set basic appearance of the element.<br>
     * @value: < default | primary | secondary | alt  >
     */
    look: { type: String, default: 'default' },
    /**
     * Set basic fill of the button.<br>
     * @value: < solid | frame >
     */
    fill: { type: String, default: 'frame' },
    /** Mark field as required (visually) */
    isRequired: { type: Boolean, default: false },
    /** Mark field as invalid (visually) */
    isError: { type: Boolean, default: false },
    /** Mark field as required (visually and practically) */
    isDisabled: { type: Boolean, default: false },
    /** Minimum value for number input */
    min: { type: [Number, String], default: '' },
    /** Maximum value for number input */
    max: { type: [Number, String], default: '' },
    /**
     * Step for number input. Used for calculating precision
     * and rounding value
     */
    step: { type: [Number, String], default: '' },
  },
  computed: {
    reListeners () {
      const { input, ...listeners } = this.$listeners
      return listeners
    },
    inputId () {
      return `ui-text__input-${this._uid}`
    },
  },
  methods: {
    input (event) {
      /**
       * Regular input with raw value.
       * @type String
       */
      this.normalizeTargetValue(event.target)
      this.$emit('input', event.target.value)
    },
    hasSlot (slot) {
      return !!this.$slots[slot]
    },
    normalizeTargetValue (target) {
      if (this.type === 'number' && target.value !== '') {
        target.value = this.normalizeDecimalPrecision(
          this.normalizeRange(target.value)
        )
      }
    },
    normalizeRange (value) {
      let result = value
      if (this.max !== '' && Number(value) > Number(this.max)) {
        result = this.max
      } else if (this.min !== '' && Number(value) < Number(this.min)) {
        result = this.min
      }
      return result
    },

    normalizeDecimalPrecision (value) {
      if (!this.step) {
        return value
      }
      let precision
      try {
        precision = String(this.step)
          .match(/(?:\.|,)\d+$/)[0]
          .slice(1).length
      } catch (error) {
        precision = 0
      }

      let result = value
      if (precision) {
        const detectRe = new RegExp(`(?:\\.|,)\\d{${precision + 1},}$`)
        if (detectRe.test(value)) {
          const replaceRe = new RegExp(`((?:\\.|,)\\d{${precision}})\\d*`)
          result = value.replace(replaceRe, '$1')
        }
      } else {
        result = String(value).split(/\.|,/)[0]
      }
      return result
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../../styles/all";

.ui-text {
  @include ui-wrap(flex);
  --ui-input-width: 560px;
  --ui-input-caret-color: #3a4180;
  --ui-input-color: #3a4180;
  --ui-input-padding: 17px 0 6px 0;
  --ui-input-border-bottom: 1px solid #9ba0d2;
  --ui-input-background: radial-gradient(rgb(58, 65, 128), rgb(58, 65, 128))
    center bottom -0.1rem / 0px 0.2rem no-repeat;
  --ui-input-transition-property: background-size, background-color;
  --ui-input-font-size: 18px;
  --ui-input-line-height: 1.25;
  --ui-input-border: none;
  --ui-input-border-color: rgba(58, 65, 128, 0.7);
  --ui-einput-color: rgb(58, 65, 128);

  --ui-label-color: #3a4180;
  --ui-label-margin-top: 0.4rem;
  --ui-label-font-size: 1.2rem;
  --ui-label-line-height: 1.25;

  flex-wrap: wrap;
  position: relative;

  &__error > &__label {
    color: var(--ui-col-error);
  }

  &__dock {
    position: absolute;
    top: 1px;
    width: var(--ui-line-height);
    height: var(--ui-line-height);
    text-align: center;

    &_left {
      left: 5px;
    }

    &_right {
      right: 5px;
    }
  }

  &__input {
    @include ui-base();
    @include ui-can-fill();
    @include ui-can-text();
    @include ui-can-disabled();
    @include ui-can-error(var(--ui-einput-color), var(--ui-col-error));

    width: var(--ui-input-width);
    caret-color: var(--ui-input-caret-color);
    color: var(--ui-input-color);
    padding: var(--ui-input-padding);
    border-bottom: var(--ui-input-border-bottom);
    background: var(--ui-input-background);
    transition-property: var(--ui-input-transition-property);
    font-size: var(--ui-input-font-size);
    line-height: var(--ui-input-line-height);
    border: var(--ui-input-border);
    border-bottom: var(--ui-input-border-bottom);
    border-color: var(--ui-input-border-color);

    display: block;
    flex-grow: 1;
    padding: var(--ui-input-padding);
    max-width: 100%;
    position: relative;
    padding: 1.5rem 0 0.6rem 0;
    font-size: 1.8rem;
    line-height: 1.25;
    padding: 1.5rem 0 0.6rem 0;

    &:placeholder-shown {
      color: transparent;
    }

    &:focus + span,
    &:not(:placeholder-shown) + span {
      top: 0;
      font-size: 1.2rem;
    }

    &[fill="none"] {
      padding-left: 0;
    }
    @include reset-number-appearance();
  }

  &__dock_right ~ &__input {
    padding-right: calc(var(--ui-line-height) * 1.1);
  }

  &__dock_left ~ &__input {
    padding-left: calc(var(--ui-line-height) * 1.1);
  }

  &__label {
    display: block;
    width: 100%;
    clear: both;
    color: var(--ui-label-color);
    margin-top: var(--ui-label-margin-top);
    font-size: var(--ui-label-font-size);
    line-height: var(--ui-label-line-height);

    &_required-star {
      @include ui-can-text();
      @include ui-can-fill();

      font-size: inherit;
      padding: var(--input-padding);
      pointer-events: none;
      visibility: visible;
      line-height: inherit;
      position: absolute;
      top: 0;
      left: 0;

      // &::after {
      //   content: "*";
      //   display: inline;
      //   color: var(--ui-col-error);
      //   margin-left: -1ch * 0.3;
      // }
    }
  }

  &__span {
    position: absolute;
    transition: all 0.2s;
    top: 1.5rem;
    font-size: 1.8rem;
    line-height: 1;
    color: var(--ui-col-text);

    &:focus {
      outline: none;
      transition: all 0.2s;
      font-size: 1.2rem;
      line-height: 1;
    }
  }

  &__input[is-required="true"]:placeholder-shown ~ &__label_required-star {
    visibility: visible;
  }
}
</style>
