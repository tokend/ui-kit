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
      :look="look"
      :fill="fill"
      :is-required="isRequired"
      :is-disabled-phone="isDisabled"
      :is-error="isError"
    >
      <ui-button
        fill="none"
        :look="look"
        class="ui-phone__country-dropdown-btn"
        :is-disabled-phone="isDisabled"
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
      :look="look"
      :fill="fill"
    >
      <ui-text
        fill="frame"
        look="secondary"
        class="ui-phone__country-dropdown-filter"
        v-model="filter"
        :is-disabled-phone="isDisabled"
        :placeholder="searchPlaceholder"
        ref="searchFilter"
      >
        <ui-button
          class="ui-phone__country-dropdown-filter-clear"
          v-if="filter"
          @click="filter = ''"
          fill="none"
          slot="dock-right"
        >
          x
        </ui-button>
      </ui-text>

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
          return item.name.toLowerCase().includes(this.filter.toLowerCase())
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
  --ui-phone-padding: 0 1rem 0 0;

  --ui-phone-border-color: var(--ui-col-primary);
  --ui-phone-border-color-focused: var(--ui-col-primary-darken);

  --ui-phone-label-text-color: var(--ui-col-primary);
  --ui-phone-label-font-size: 0.75em;

  --ui-phone-border-top: none;
  --ui-phone-border-left: none;
  --ui-phone-border-right: none;
  --ui-phone-border-bottom: 1px solid var(--ui-phone-border-color);

  --ui-phone-button-text-color: var(--ui-col-secondary);
  --ui-phone-dropdown-shadow: var(--ui-box-shadow);
  --ui-phone-err-msg-font-size: 0.75em;
  --ui-phone-err-msg-padding-top: 0.4rem;

  --ui-phone-err-background: transparent;
  --ui-phone-err-color: var(--ui-phone-button-text-color);
  @include ui-wrap(flex);

  flex-wrap: wrap;
  position: relative;

  &__label {
    display: block;
    width: 100%;

    color: var(--ui-phone-label-text-color);
    font-size: var(--ui-phone-label-font-size);
    line-height: var(--ui-phone-label-font-size);

    &--error {
      color: var(--ui-col-error);
    }
  }

  &__input-wrp {
    @include ui-base(flex);
    @include ui-can-fill();
    @include ui-can-text();
    @include ui-can-disabled(var(--ui-phone-err-background),none, var(--ui-phone-err-color));
    @include ui-can-error();
    border: var(--ui-phone-border-top);
    border-left: var(--ui-phone-border-left);
    border-right: var(--ui-phone-border-right);
    border-bottom: var(--ui-phone-border-bottom);

    &--focused {
      border-color: var(--ui-phone-border-color-focused) !important;
    }

    padding: var(--ui-phone-padding);
    width: 100%;

    @include respond-below(xs) {
      max-height: var(--ui-line-height);
    }
  }

  &__country-dropdown-btn {
    display: flex;
    align-items: center;
    white-space: nowrap;

    & /deep/ .ui-button {
      display: flex;
      align-content: center;

      &__button {
        color: var(--ui-phone-button-text-color);

        position: relative;
        height: var(--ui-line-height);
        display: block;
        padding: 0 0.5em 0 38px;
        text-align: left;
      }
    }

    & /deep/ .ui-flag {
      position: absolute;
      left: 0;
      top: 50%;
      border-radius: 3px;
      margin: 0 0.25em 0 0.25em;
      transform: translateY(-50%);
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

    @include ui-can-fill();

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
    @include ui-can-fill();
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
  }

  // HACK: Remove input autofill background
  input:-webkit-autofill,
  input:-webkit-autofill:hover,
  input:-webkit-autofill:focus,
  input:-webkit-autofill:active {
    -webkit-box-shadow: 0 -5.5em 0 0 var(--ui-col-bg) inset !important;
  }

  &__input-wrp[fill="solid"] {
    &[look="primary"] {
      color: var(--ui-col-primary-contrast);
    }

    &[look="secondary"] {
      color: var(--ui-col-secondary-contrast);
    }

    &[is-error] {
      color: var(--ui-col-error);
    }

    &[is-disabled-phone] {
      color: var(--ui-col-disabled);
    }
  }

  &__error-msg {
    color: var(--ui-col-error);
    font-size: var(--ui-phone-err-msg-font-size);
    line-height: var(--ui-phone-err-msg-font-size);
    padding-top: var(--ui-phone-err-msg-padding-top);
  }
}
</style>
