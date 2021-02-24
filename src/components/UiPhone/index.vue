<template>
  <div
    class="ui-phone"
    :class="{ 'ui-phone_is-open': isOpenCountries }"
  >
    <span
      v-if="hasSlot('default')"
      :class="{
        'ui-phone__label': true,
        'ui-phone__label--error': isError,
      }"
    >
      <!-- @slot Use this slot to place the text field label content -->
      <slot />
    </span>

    <div
      :class="{
        'ui-phone__input-wrp': true,
        'ui-phone__input-wrp--focused': isFocused,
      }"
      :is-required="isRequired"
      :is-disabled-phone="isDisabled"
      :is-error="isError"
    >
      <ui-button
        fill="none"
        look="secondary"
        class="ui-phone__country-dropdown-btn"
        :is-disabled="isDisabled"
        @click="toggleCountriesDropdown"
      >
        <ui-flag :country="countryCode" />
        {{ dialCode }}
      </ui-button>

      <the-mask
        class="ui-phone__input"
        v-model="phoneNumber"
        :mask="currentMask"
        type="text"
        :is-required="isRequired"
        :is-disabled-phone="isDisabled"
        :is-error="isError"
        :disabled="isDisabled"
        :placeholder="placeholder"
        @input="input"
        @focus.native="$emit('focus') && focus()"
        @blur.native="$emit('blur') && blur()"
        v-on="reListeners"
        v-bind="$attrs"
        :masked="false"
      />
    </div>
    <div
      class="ui-phone__country-dropdown"
      v-if="isOpenCountries"
      v-on-clickaway="handleClickawayDirective"
    >
      <ui-text
        class="ui-phone__country-dropdown-filter"
        v-model="filter"
        :is-disabled-phone="isDisabled"
        :placeholder="searchPlaceholder"
        ref="searchFilter"
      />

      <div class="ui-phone__country-dropdown-scroll">
        <ui-button
          class="ui-phone__country-dropdown-item"
          fill="none"
          look="default"
          @click="handleCountrySelect(country)"
          v-for="country in countryPhones"
          :key="country.code"
        >
          <ui-flag
            :class="`ui-phone__country-dropdown-item-flag`"
            :country="country.countryCode"
          />
          <div class="ui-phone__country-dropdown-item-country">
            {{ country.label }}
          </div>
        </ui-button>
        <div
          class="ui-phone__country-dropdown-item-nothing"
          v-if="!countryPhones.length"
        >
          {{ searchNotFoundMessage }}
        </div>
      </div>
    </div>

    <span
      class="ui-phone__error-msg"
      v-if="hasSlot('error') && isError"
    >
      <!-- @slot Content of this slot will be visible if ``:is-error`` true -->
      <slot name="error" />
    </span>
  </div>
</template>

<script>
import { TheMask } from 'vue-the-mask'
import {
  UiFlag,
  UiText,
  UiButton,
} from '../../index'

import { directive as onClickaway } from 'vue-clickaway'

import phonesData from '../../utils/countries'

const MASK = '### ## ## ###'

export default {
  name: 'UiPhone',
  components: {
    TheMask,
    UiFlag,
    UiText,
    UiButton,
  },
  directives: {
    onClickaway
  },
  inheritAttrs: false,
  props: {
    /** Regular value */
    value: { type: String, default: '' },
    /** Regular placeholder */
    placeholder: { type: String, default: MASK.split('#').join('-') },
    /** Mark field as required (visually) */
    isRequired: { type: Boolean, default: false },
    /** Mark field as invalid (visually) */
    isError: { type: Boolean, default: false },
    /** Mark field as required (visually and practically) */
    isDisabled: { type: Boolean, default: false },
    /** The ISO codes of the countries that are pinned in the countries list */
    pinnedCountries: { type: Array, default: () => ['ua'] },
    /** Display only pinned countries in the country codes list */
    filterCountries: { type: Boolean, default: false },
    /** Placeholder for searching countries */
    searchPlaceholder: { type: String, default: 'Search' },
    searchNotFoundMessage: { type: String, default: 'No countries found' }
  },

  data: () => {
    return {
      filter: '',
      phoneNumber: '',
      currentMask: MASK,
      dialCode: '',
      countryCode: '',
      isOpenCountries: false,
      isFocused: false,
    }
  },

  computed: {
    reListeners () {
      const { input, ...listeners } = this.$listeners
      return listeners
    },

    phones () {
      return Object.values(phonesData).filter(phone => {
        return this.filterCountries
          ? this.pinnedCountries.includes(phone.code)
          : true
      })
    },

    countryPhones () {
      const pinned = this.pinnedCountries

      const pinnedPhones = this.phones
        .filter(item => pinned.includes(item.code))
        .sort((a, b) => pinned.indexOf(a.code) - pinned.indexOf(b.code))
      const restPhones = this.filterCountries
        ? []
        : this.phones.filter(item => !this.pinnedCountries.includes(item.code))

      return [...pinnedPhones, ...restPhones]
        .filter(item => {
          return item.name.toLowerCase().includes(this.filter.toLowerCase()) ||
            item.code.includes(this.filter.toLowerCase()) ||
            `+${item.dialCode}`.includes(this.filter)
        })
        .map(item => ({
          value: `+${item.dialCode}`,
          label: `+${item.dialCode} ${item.name}`,
          selectedLabel: `+${item.dialCode}`,
          countryCode: item.code
        }))
    }
  },

  watch: {
    isOpenCountries (val) {
      if (val) {
        this.$nextTick(() => this.$refs.searchFilter.$el.focus())
      } else {
        this.filter = ''
      }
    },
    dialCode: {
      immediate: true,
      handler (value) {
        for (let code in this.phones) {
          if (value.endsWith(this.phones[code].dialCode.toString())) {
            this.currentMask = this.phones[code].mask
            if (!this.countryCode) this.countryCode = this.phones[code].code
            this.$emit(
              'input',
              this.phoneNumber ? `${value}${this.phoneNumber}` : ''
            )
            return
          }
        }
      }
    },

    value (phone) {
      this.updatePhoneNumber(phone)
    }
  },

  created () {
    this.updatePhoneNumber(this.value)

    if (!this.value) {
      const phone =
        this.phones.find(item => item.code === this.pinnedCountries[0]) ||
        this.phones[0]

      this.dialCode = `+${phone.dialCode}`
      this.currentMask = phone.mask
    }
  },

  methods: {
    focus () {
      this.isFocused = true
    },
    blur () {
      this.isFocused = false
    },
    handleCountrySelect (country) {
      this.closeCountries()
      this.countryCode = country.countryCode
      this.dialCode = country.value
    },
    handleClickawayDirective () {
      this.handleClickAway()
    },
    closeCountries () {
      this.isOpenCountries = false
    },
    toggleCountriesDropdown () {
      this.isOpenCountries = !this.isOpenCountries
    },
    handleClickAway () {
      if (this.isOpenCountries) this.closeCountries()
    },
    input (value) {
      // /**
      //  * Triggers when the number changes
      //  * @property {string} value - the whole phone number.
      //  */
      this.$emit('input', value ? `${this.dialCode}${value}` : '')
    },
    hasSlot (slot) {
      return !!this.$slots[slot]
    },

    updatePhoneNumber (val) {
      if (!val) {
        this.phoneNumber = ''
      }

      const phone = this.phones.find(p => {
        const exampleNumber = p.example.replace(/\D/g, '')
        const codelessNumber = val.replace(`+${p.dialCode}`, '')

        return (
          val.startsWith(`+${p.dialCode}`) &&
          codelessNumber.length === exampleNumber.length
        )
      })

      if (phone) {
        this.dialCode = `+${phone.dialCode}`
        this.currentMask = phone.mask
        this.phoneNumber = val.replace(`+${phone.dialCode}`, '')
        if (this.dialCode !== `+${phone.dialCode}`) {
          this.countryCode = phone.code
        }
      }
    }
  }
}

/* eslint-enable object-curly-newline, no-unused-vars */
</script>

<style lang="scss" scoped>
@import "../../styles/all";

.ui-phone {
  --ui-phone-background: #fff;
  --ui-phone-font-size: var(--ui-font-size);

  --ui-phone-padding-right: 1.2rem;
  --ui-phone-padding-left: 1.2rem;
  --ui-phone-padding-bottom: 0;
  --ui-phone-padding-top: 0;
  --ui-phone-gap: 0.75rem;

  --ui-phone-border-color: var(--ui-col-secondary);
  --ui-phone-border-color-hover: var(--ui-phone-border-color);
  --ui-phone-border-color-focused: var(--ui-phone-border-color);
  --ui-phone-border-color-error: var(--ui-phone-border-color);
  --ui-phone-border-color-error–hover: var(--ui-phone-border-color-hover);
  --ui-phone-border-color-error–focused: var(--ui-phone-border-color-focused);

  --ui-phone-border-style: solid;
  --ui-phone-border-top-width: 1px;
  --ui-phone-border-bottom-width: 1px;
  --ui-phone-border-left-width: 1px;
  --ui-phone-border-right-width: 1px;
  --ui-phone-border-top:
    var(--ui-phone-border-top-width)
    var(--ui-phone-border-style)
    var(--ui-phone-border-color)
  ;
  --ui-phone-border-left:
    var(--ui-phone-border-left-width)
    var(--ui-phone-border-style)
    var(--ui-phone-border-color)
  ;
  --ui-phone-border-right:
    var(--ui-phone-border-right-width)
    var(--ui-phone-border-style)
    var(--ui-phone-border-color)
  ;
  --ui-phone-border-bottom:
    var(--ui-phone-border-bottom-width)
    var(--ui-phone-border-style)
    var(--ui-phone-border-color)
  ;
  --ui-phone-border-radius: var(--ui-border-radius);

  --ui-phone-box-shadow: none;
  --ui-phone-box-shadow-hover: none;
  --ui-phone-box-shadow-focused: 0 0 0 1px var(--ui-col-primary);
  --ui-phone-box-shadow-error: --ui-phone-box-shadow;
  --ui-phone-box-shadow-error-hover: var(--ui-phone-box-shadow-hover);
  --ui-phone-box-shadow-error-focused: var(--ui-phone-box-shadow-focused);

  --ui-phone-label-background: var(--ui-phone-background);
  --ui-phone-label-padding: 0 0.25rem;
  --ui-phone-label-color: var(--ui-col-primary);
  --ui-phone-label-color-error: var(--ui-phone-label-color);
  --ui-phone-label-font-size: 0.75em;
  --ui-phone-label-top: calc(var(--ui-phone-label-font-size) / -2);;
  --ui-phone-label-left: calc(
    var(--ui-phone-padding-left) +
    var(--ui-phone-border-left-width)
  );

  --ui-phone-button-text-color: var(--ui-col-secondary);
  --ui-phone-dropdown-shadow: var(--ui-box-shadow);
  --ui-phone-err-msg-font-size: 0.75em;
  --ui-phone-err-msg-padding-top: 0.4rem;

  --ui-phone-required-star-color: var(--ui-col-error);

  @include ui-wrap(flex);

  flex-wrap: wrap;
  position: relative;

  &__label {
    display: block;
    width: auto;

    color: var(--ui-phone-label-color);
    font-size: var(--ui-phone-label-font-size);
    line-height: var(--ui-phone-label-font-size);
    background: var(--ui-phone-label-background);
    padding: var(--ui-phone-label-padding);

    position: absolute;
    top: var(--ui-phone-label-top);
    left: var(--ui-phone-label-left);
  }

  &__input-wrp {
    @include ui-base(flex);
    @include ui-can-text();
    @include ui-can-disabled();

    display: grid;
    grid-auto-flow: column;
    grid-template-columns: min-content 1fr;
    gap: var(--ui-phone-gap);
    width: 100%;
    padding: (
      var(--ui-phone-padding-top)
      var(--ui-phone-padding-right)
      var(--ui-phone-padding-bottom)
      var(--ui-phone-padding-left)
    );

    border-top: var(--ui-phone-border-top);
    border-left: var(--ui-phone-border-left);
    border-right: var(--ui-phone-border-right);
    border-bottom: var(--ui-phone-border-bottom);
    border-radius: var(--ui-phone-border-radius);
    box-shadow: var(--ui-phone-box-shadow);

    &, & /deep/ input {
      font-size: var(--ui-phone-font-size);
      line-height: var(--ui-phone-font-size);
    }

    &:hover {
      border-color: var(--ui-phone-border-color-hover);
      box-shadow: var(--ui-phone-box-shadow-hover);
    }

    &--focused {
      &, &:hover {
        border-color: var(--ui-phone-border-color-focused);
        box-shadow: var(--ui-phone-box-shadow-focused);
      }
    }

    &[is-error] {
      box-shadow: var(--ui-phone-box-shadow-error);
      border-color: var(--ui-phone-border-color-error);

      &:hover {
        border-color: var(--ui-phone-border-color-error–hover);
        box-shadow: var(--ui-phone-box-shadow-error-hover);
      }

      .ui-phone__input {
        @include placeholder() {
          color: var(--ui-phone-label-color-error);
        }
      }
    }

    &--focused[is-error] {
      &, &:hover {
        border-color: var(--ui-phone-border-color-error–focused);
        box-shadow: var(--ui-phone-box-shadow-error-focused);
      }
    }

    &[is-error] ~ .ui-phone__label {
      color: var(--ui-phone-label-color-error);
    }

    &[is-required] ~ .ui-phone__label::after {
      color: var(--ui-phone-required-star-color);
      content: '*';
      position: relative;
    }
  }

  &__country-dropdown-btn {
    --ui-button-width: auto !important;
    --ui-button-line-height: var(--ui-phone-font-size) !important;

    display: flex;
    align-items: center;
    white-space: nowrap;

    & /deep/ .ui-button {
      display: flex;
      align-content: center;
    }

    & /deep/ .ui-flag {
      // TODO: vars:
      border-radius: 3px;
      box-shadow: 0 0 2px rgba(0, 0, 0, 0.3);
    }
  }

  &__country-dropdown {
    z-index: 1;
    position: absolute;
    top: var(--ui-line-height);
    left: 0;
    right: 0;
    padding: 1em;
    background: var(--ui-col-bg);
    box-shadow: var(--ui-phone-dropdown-shadow);

    @include respond-below(sm) {
      position: static;
      top: unset;
      left: unset;
      right: unset;
      margin: 0;
      width: 100%;
    }

    &-scroll {
      max-height: 25vh;
      overflow-x: hidden;
      overflow-y: auto;

      @include scrollbar-awesome();

      @include respond-below(sm) {
        max-height: 45vh;
        min-height: 6em;
      }
    }

    &-filter {
      margin-bottom: 1.5em;

      &-clear {
        & /deep/ .ui-button__button {
          padding-left: 0;
          padding-right: 0;
        }
      }
    }

    &-item {
      display: block;
      line-height: var(--ui-line-height);
      min-height: var(--ui-line-height);

      & /deep/ .ui-button__button {
        line-height: calc(var(--ui-line-height) / 2);
        min-height: var(--ui-line-height);
        display: flex;
        justify-content: left;
        align-items: center;
        padding-left: 0;
        padding-right: 0;
        border-radius: 0 !important;
      }

      &-flag {
        flex: 0;
        min-width: 27px;
        margin-right: 0.5em;
      }

      &-country {
        flex: 1;
        text-align: left;
        color: var(--ui-col-dark);
      }

      &-nothing {
        text-align: center;
        padding: 2em 0;
        color: var(--ui-col-disabled);
      }

      &:hover &-country {
        color: var(--ui-col-dark);
      }
    }
  }

  &_is-open &__input-wrp {
    border-bottom-left-radius: 0 !important;
    border-bottom-right-radius: 0 !important;
    border-bottom-color: var(--ui-col-light) !important;
  }

  &_is-open &__country-dropdown {
    border-top-left-radius: 0 !important;
    border-top-right-radius: 0 !important;
    border-top-color: var(--ui-col-light) !important;
  }

  &__input {
    @include ui-can-text();

    display: block;
    outline: none;
    border: none;
    width: 100%;
    padding: 0 1.5em 0 0;
    background-color: transparent;
    color: inherit;
    border-radius: 0 !important;

    @include respond-below(xs) {
      margin-right: 0.4em;
    }

    @include placeholder() {
      color: var(--ui-phone-label-color);
    }
  }

  // HACK: Remove input autofill background
  input:-webkit-autofill,
  input:-webkit-autofill:hover,
  input:-webkit-autofill:focus,
  input:-webkit-autofill:active {
    -webkit-box-shadow: 0 -5.5em 0 0 var(--ui-col-bg) inset !important;
  }

  &__error-msg {
    color: var(--ui-col-error);
    font-size: var(--ui-phone-err-msg-font-size);
    line-height: var(--ui-phone-err-msg-font-size);
    padding-top: var(--ui-phone-err-msg-padding-top);
  }
}
</style>
