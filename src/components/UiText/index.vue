<template>
  <label
    class="ui-text"
    :for="inputId"
    :is-disabled="isDisabled"
  >
    <input
      class="ui-text__input"
      :id="inputId"
      :is-required="isRequired"
      :is-disabled="isDisabled"
      :is-error="isError"
      :disabled="isDisabled"
      :placeholder="' '"
      :value="value"
      :type="type"
      @input="input"
      v-on="reListeners"
      v-bind="$attrs"
    >

    <span
      class="ui-text__label"
      v-if="hasSlot('default')"
    >
      <!-- @slot Use this slot to place the text field label content -->
      <slot />
    </span>

    <span
      v-if="hasSlot('error') && isError"
      class="ui-text__error-msg"
      :is-error="isError"
    >
      <!-- @slot Content of this slot will be visible if ``:is-error`` true -->
      <slot name="error" />
    </span>
  </label>
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
  --ui-text-color: var(--ui-col-primary);
  --ui-text-color-error: var(--ui-text-color);
  --ui-text-font-size: var(--ui-font-size);
  --ui-text-background: #fff;

  --ui-text-padding-left: 1.2rem;
  --ui-text-padding-right: 1.2rem;
  --ui-text-padding-top: 1rem;
  --ui-text-padding-bottom: 1rem;

  --ui-text-border-color: var(--ui-col-secondary);
  --ui-text-border-color-hover: var(--ui-text-border-color);
  --ui-text-border-color-focused: var(--ui-text-border-color);
  --ui-text-border-color-error: var(--ui-text-border-color);
  --ui-text-border-color-error–hover: var(--ui-text-border-color-hover);
  --ui-text-border-color-error–focused: var(--ui-text-border-color-focused);

  --ui-text-border-top-width: 1px;
  --ui-text-border-bottom-width: 1px;
  --ui-text-border-left-width: 1px;
  --ui-text-border-right-width: 1px;

  --ui-text-border-style: solid;

  --ui-text-border-top:
    var(--ui-text-border-top-width)
    var(--ui-text-border-style)
    var(--ui-text-border-color)
  ;
  --ui-text-border-left:
    var(--ui-text-border-left-width)
    var(--ui-text-border-style)
    var(--ui-text-border-color)
  ;
  --ui-text-border-right:
    var(--ui-text-border-right-width)
    var(--ui-text-border-style)
    var(--ui-text-border-color)
  ;
  --ui-text-border-bottom:
    var(--ui-text-border-bottom-width)
    var(--ui-text-border-style)
    var(--ui-text-border-color)
  ;

  --ui-text-border-radius: var(--ui-border-radius);

  --ui-text-label-background: var(--ui-text-background);
  --ui-text-label-padding: 0;
  --ui-text-label-color: var(--ui-col-secondary);
  --ui-text-label-color-error: var(--ui-text-label-color);
  --ui-text-label-color-focused: var(--ui-col-primary);
  --ui-text-label-color-filled: var(--ui-col-primary);
  --ui-text-label-font-size: var(--ui-text-font-size);
  --ui-text-label-transition-duration: 300ms;
  --ui-text-label-top: calc(
    var(--ui-text-padding-top) +
    var(--ui-text-border-top-width) +
    var(--ui-text-border-bottom-width)
  );
  --ui-text-label-left: calc(
    var(--ui-text-padding-left) +
    var(--ui-text-border-left-width)
  );

  --ui-text-label-padding-focused: 0 0.25rem;
  --ui-text-label-left-focused: var(--ui-text-label-left);
  --ui-text-label-font-size-focused: 1.2rem;
  --ui-text-label-top-focused: calc(
    var(--ui-text-label-font-size-focused) / -2
  );

  --ui-text-label-padding-filled: var(--ui-text-label-padding-focused);
  --ui-text-label-left-filled: var(--ui-text-label-left-focused);
  --ui-text-label-font-size-filled: var(--ui-text-label-font-size-focused);
  --ui-text-label-top-filled: var(--ui-text-label-top-focused);

  --ui-text-box-shadow: none;
  --ui-text-box-shadow-hover: none;
  --ui-text-box-shadow-focused: 0 0 0 1px var(--ui-col-primary);
  --ui-text-box-shadow-error: var(--ui-text-box-shadow);
  --ui-text-box-shadow-error-hover: var(--ui-text-box-shadow-hover);
  --ui-text-box-shadow-error-focused: var(--ui-text-box-shadow-focused);

  --ui-text-err-msg-font-size: 0.75em;
  --ui-text-err-msg-padding-top: 0.75rem;

  --ui-text-required-star-color: var(--ui-col-error);

  @include ui-wrap(flex);
  @include ui-can-disabled(0.5);

  flex-wrap: wrap;
  position: relative;

  &__input {
    color: var(--ui-text-color);
    background: var(--ui-text-background);
    display: block;
    flex-grow: 1;
    max-width: 100%;

    @include ui-base();
    @include ui-can-text();
    @include reset-number-appearance();

    font-size: var(--ui-text-font-size);
    line-height: var(--ui-text-font-size);

    border-top: var(--ui-text-border-top);
    border-bottom: var(--ui-text-border-bottom);
    border-left: var(--ui-text-border-left);
    border-right: var(--ui-text-border-right);
    border-radius: var(--ui-text-border-radius);

    padding: (
      var(--ui-text-padding-top)
      var(--ui-text-padding-right)
      var(--ui-text-padding-bottom)
      var(--ui-text-padding-left)
    );

    box-shadow: var(--ui-text-box-shadow);

    &:hover {
      border-color: var(--ui-text-border-color-hover);
      box-shadow: var(--ui-text-box-shadow-hover);
    }

    &:focus {
      border-color: var(--ui-text-border-color-focused);
      box-shadow: var(--ui-text-box-shadow-focused);
    }

    & ~ .ui-text__error-msg {
      display: block;
      width: 100%;

      color: var(--ui-col-error);
      font-size: var(--ui-text-err-msg-font-size);
      line-height: var(--ui-text-err-msg-font-size);
      padding-top: var(--ui-text-err-msg-padding-top);
    }

    &[is-required] ~ .ui-text__label::after {
      color: var(--ui-text-required-star-color);
      content: '*';
      position: relative;
    }

    &[is-error] {
      box-shadow: var(--ui-text-box-shadow-error);
      border-color: var(--ui-text-border-color-error);
      color: var(--ui-text-color-error);

      &:hover {
        border-color: var(--ui-text-border-color-error–hover);
        box-shadow: var(--ui-text-box-shadow-error-hover);
      }

      &:focus {
        border-color: var(--ui-text-border-color-error–focused);
        box-shadow: var(--ui-text-box-shadow-error-focused);
      }
    }

    &[is-error] ~ .ui-text__label {
      color: var(--ui-text-label-color-error);
    }
  }

  &__label {
    display: block;
    width: auto;

    pointer-events: none;
    white-space: nowrap;
    max-width: 100%;
    overflow: hidden;
    text-overflow: ellipsis;

    position: absolute;

    top: var(--ui-text-label-top);
    left: var(--ui-text-label-left);
    background: var(--ui-text-label-background);
    color: var(--ui-text-label-color);
    font-size: var(--ui-text-label-font-size);
    line-height: var(--ui-text-label-font-size);
    padding: var(--ui-text-label-padding);
    transition: var(--ui-text-label-transition-duration);

    .ui-text__input:focus ~ & {
      color: var(--ui-text-label-color-focused);
      padding: var(--ui-text-label-padding-focused);
      font-size: var(--ui-text-label-font-size-focused);
      line-height: var(--ui-text-label-font-size-focused);
      top: var(--ui-text-label-top-focused);
      left: var(--ui-text-label-left-focused);
    }

    .ui-text__input:not(:placeholder-shown) ~ & {
      color: var(--ui-text-label-color-filled);
      padding: var(--ui-text-label-padding-filled);
      font-size: var(--ui-text-label-font-size-filled);
      line-height: var(--ui-text-label-font-size-filled);
      top: var(--ui-text-label-top-filled);
      left: var(--ui-text-label-left-filled);
    }
  }
}
</style>
