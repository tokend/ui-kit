<template>
  <div class="ui-select">
    <div
      class="ui-select__wrap"
      :class="isOpen ? 'ui-select_is-open' : ''"
      v-on-clickaway="handleClickawayDirective"
      ref="selectWrap"
    >
      <ui-button
        @click="toggle"
        ref="input"
        v-model="filter"
        type="button"
        class="ui-select__button"
        :look="look"
        :fill="fill"
        :is-required="isRequired"
        :is-disabled="isDisabled"
        :is-error="isError"
        :class="{ 'ui-select__button_is-open': isOpen }"
      >
        <span
          v-if="hasSlot('dock-left')"
          class="ui-select__dock ui-select__dock_left">
          <slot name="dock-left" />
        </span>
        <img
          v-if="useIcon && currentItem"
          class="ui-select__button-icon"
          :src="currentItem.icon"
        >

        <span
          class="ui-select__button-lbl"
          :class="{
            'ui-select__button-lbl_placeholder': !value,
            'ui-select__button-lbl_placeholder-required': !value && isRequired
          }"
        >
          {{ currentLabel }}
        </span>

        <ui-icon
          icon="dropdown"
          v-if="!isDisabled"
          class="ui-select__dock ui-select__dock_caret"
          :class="{ 'ui-select__dock_caret_open': isOpen }"
        />
      </ui-button>

      <div
        class="ui-select__dropdown"
        ref="dropdown"
        :look="look"
        :fill="fill"
        :class="{
          'ui-select__dropdown_occupies-space': occupiesSpace,
          'ui-select__dropdown_sticky-filter': stickyFilter
        }"
      >
        <ui-text
          v-if="hasFilter"
          ref="input"
          v-model="filter"
          type="text"
          class="ui-select__input"
          :class="{ 'ui-select__input_sticky': stickyFilter }"
          :is-disabled="isDisabled"
          :is-error="isError"
          :disabled="isDisabled"
          :placeholder="searchPlaceholder"
        >
          <div slot="dock-left">
            <ui-icon icon="search" />
          </div>
          <div slot="dock-right">
            <a
              @click="filter = ''"
              v-if="filter">
              <ui-icon icon="close" />
            </a>
          </div>
        </ui-text>

        <div
          class="ui-select__options"
          :class="{ 'ui-select__options_sticky-filter': stickyFilter }"
        >
          <a
            v-if="nullOption"
            :class="{
              'ui-select__root-item': true,
              'ui-select__root-item_active': value === nullOptionValue
            }"
            @click.stop="handleSelect(null)"
            tabindex="0"
          >
            {{ nullOptionLabel }}
          </a>
          <div>
            <ui-select-options
              :items="localOptions"
              :current-value="value"
              :selected-value="selectedValue"
              @option-select="handleSelect"
              @nested-toggle="handleNestedToggle"
              :allow-select-group="allowSelectGroup"
            />

            <span
              class="ui-select__root-item ui-select__root-item-empty"
              @click="filter = ''"
            >
              {{ emptyMessage }}
            </span>
          </div>
        </div>
      </div>
    </div>
    <span
      class="ui-select__label"
      :class="{ 'ui-select__label_is-error': isError }"
      v-if="hasSlot('default') && (!hasSlot('error') || !isError)"
    >
      <!-- @slot Use this slot to place the text field label content -->
      <slot />
    </span>
    <span
      class="ui-select__label ui-select__label-error"
      :class="{ 'ui-select__label_is-error': isError }"
      v-if="hasSlot('error') && isError"
    >
      <!-- @slot Show this slot if ``:is-error`` true -->
      <slot name="error" />
    </span>
  </div>
</template>

<script>
import { directive as onClickaway } from 'vue-clickaway'
import VueScrollTo from 'vue-scrollto'

import { UiText, UiButton } from '../..'
import UiSelectOptions from './UiSelectOptions'
import { keyCodes } from '../../utils/keyCodes'

export default {
  components: {
    UiSelectOptions,
    UiButton,
    UiText
  },
  directives: {
    onClickaway
  },
  props: {
    /**
     *  List of items.<br>
     *  Can contain list of strings or structured items
     *  like ``{value:<String|Number>, label:<String>, selectedLabel:<String>,
     *  icon:<String> }``
     */
    options: {
      type: Array,
      default: () => []
    },
    /**
     * Filter method (default: contains): < starts | contains | default >
     */
    filterMethod: {
      type: String,
      default: 'contains',
      validator (val) {
        return ['starts', 'contains', 'default', 'none'].includes(val)
      }
    },
    value: { type: [String, Number, Object], default: null },
    emptyMessage: { type: String, default: 'No items found' },
    placeholder: { type: String, default: 'Select an item' },
    searchPlaceholder: { type: String, default: 'Search' },
    hasFilter: { type: Boolean, default: false },
    /** Set basic appearance of the element.<br>
     * @value: < default | primary | secondary | alt >
     */
    look: { type: String, default: 'default' },
    /**
     * Set basic fill of the button.<br>
     * @value: < solid | frame >
     */
    fill: { type: String, default: 'none' },
    /** Mark field as required (visually) */
    isRequired: { type: Boolean, default: false },
    /** Mark field as invalid (visually) */
    isError: { type: Boolean, default: false },
    /** Mark field as required (visually and practically) */
    isDisabled: { type: Boolean, default: false },
    /** Label for null option. Set it to make user able to "deseltct" */
    nullOption: { type: [Boolean, String, Object], default: null },
    occupiesSpace: { type: Boolean, default: false },
    useIcon: { type: Boolean, default: false },
    allowSelectGroup: { type: Boolean, default: false },
    /** Use hot keys while navigating through the dropdown */
    useHotKeys: { type: Boolean, default: false },
    /**
     * Automatically manage clickaway when dropdown is opened.
     */
    clickaway: { type: Boolean, default: true },
    /**
     * Pin filter input to the top of dropdown.
     */
    stickyFilter: { type: Boolean, default: false },
  },

  data () {
    return {
      filter: '',
      isOpen: false,
      currentItem: null,
      selectedIndex: -1,
      localOptions: []
    }
  },

  computed: {
    flattenOptions () {
      return this.flattenShownOptions(this.localOptions)
    },

    selectedItem () {
      return this.flattenOptions[this.selectedIndex]
    },

    selectedValue () {
      return this.useHotKeys && this.selectedItem
        ? this.selectedItem.value
        : null
    },

    currentLabel () {
      if (!this.currentItem) return this.placeholder
      return this.currentItem.selectedLabel
        ? this.currentItem.selectedLabel
        : this.currentItem.label || this.nullOptionValue
    },

    nullOptionValue () {
      if (this.nullOption === null) return null
      if (typeof this.nullOption === 'string') return null
      if (typeof this.nullOption === 'object') return this.nullOption.value
      return null
    },

    nullOptionLabel () {
      if (this.nullOption === null || typeof this.nullOption === 'boolean') {
        return this.placeholder
      }
      if (typeof this.nullOption === 'string') {
        return this.nullOption || this.placeholder
      }
      if (typeof this.nullOption === 'object') {
        return this.nullOption.label || this.placeholder
      }
      return this.placeholder
    }
  },

  watch: {
    filter: {
      immediate: true,
      handler () {
        this.$emit('filter', this.filter)
        this.filterLocalOptions(this.localOptions)
        this.selectedIndex = this.flattenOptions.indexOf(this.currentItem)
      }
    },

    value: {
      immediate: true,
      handler (val) {
        // find item by value
        this.$set(
          this,
          'currentItem',
          val ? this.findItemByVal(val, this.localOptions) : null
        )
      }
    },

    options: {
      immediate: true,
      handler (options) {
        this.$set(this, 'localOptions', this.generateLocalOptions(options))
        this.filterLocalOptions(this.localOptions)
        // if options has changed, find item by value again,
        // and set null if not found
        this.$set(
          this,
          'currentItem',
          this.findItemByVal(this.value, this.localOptions)
        )
      }
    }
  },

  beforeDestroy () {
    if (this.useHotKeys) {
      this.$refs.selectWrap.removeEventListener('keydown', this.handleKeyDown)
    }
  },

  methods: {
    filterLocalOptions (items) {
      if (this.filterMethod === 'none') {
        return
      }

      let found = 0

      for (let i in items) {
        let str = items[i].label
        items[i].show = false

        switch (this.filterMethod) {
          case 'starts':
            if (str.toLowerCase().startsWith(this.filter.toLowerCase())) {
              items[i].show = true
              found++
            }
            break
          case 'contains':
          default:
            if (str.toLowerCase().indexOf(this.filter.toLowerCase()) >= 0) {
              items[i].show = true
              found++
            }
            break
        }

        if (items[i].children) {
          if (this.filterLocalOptions(items[i].children)) items[i].show = true
        }
      }
      return found
    },

    generateLocalOptions (source, target = []) {
      for (let i in source) {
        target[i] = {
          label: (source[i].label || source[i].value || source[i]).toString(),
          value: source[i].value || source[i].label || source[i],
          icon: source[i].icon,
          selectedLabel: (source[i].selectedLabel || '').toString(),
          show: true,
          expand: false,
          hidden: source[i].hidden || false,
        }

        if (source[i].children) {
          target[i].children = this.generateLocalOptions(
            source[i].children,
            target[i].children
          )
        }
      }
      return target
    },

    handleClickawayDirective () {
      this.handleClickAway()
    },

    handleClickAway () {
      if (this.clickaway && this.isOpen) this.close()
    },

    open () {
      if (this.isDisabled) return

      this.isOpen = true
      this.selectedIndex = this.flattenOptions.indexOf(this.currentItem)

      if (this.useHotKeys) {
        this.$refs.selectWrap.addEventListener('keydown', this.handleKeyDown)
      }

      if (this.hasFilter) {
        this.$refs.input.$el.focus()
      }
    },

    close () {
      this.isOpen = false

      if (this.useHotKeys) {
        this.$refs.selectWrap.removeEventListener('keydown', this.handleKeyDown)
      }

      if (this.hasFilter) {
        this.$refs.input.$el.blur()
      }
    },

    toggle () {
      this.isOpen ? this.close() : this.open()
    },

    flattenShownOptions (options) {
      return options.filter(option => option.show)
        .reduce((list, item) => [
          ...list,
          item,
          ...this.flattenShownOptions(item.expand ? item.children || [] : [])
        ], [])
    },

    handleKeyDown (e) {
      if (Object.values(keyCodes).includes(e.keyCode)) {
        e.preventDefault()

        const itemsCount = this.flattenOptions.length

        switch (e.keyCode) {
          case keyCodes.arrowUp:
            this.selectedIndex = this.selectedIndex <= 0
              ? itemsCount - 1
              : this.selectedIndex - 1
            break

          case keyCodes.arrowDown:
            this.selectedIndex = this.selectedIndex >= itemsCount - 1
              ? 0
              : this.selectedIndex + 1
            break

          case keyCodes.arrowRight:
            if (!this.selectedItem.expand) {
              this.handleNestedToggle(this.selectedItem)
            }
            break

          case keyCodes.arrowLeft:
            if (this.selectedItem.expand) {
              this.handleNestedToggle(this.selectedItem)
            }
            break

          case keyCodes.escape:
            this.close()
            break

          case keyCodes.enter:
            if (this.selectedIndex !== -1) {
              this.handleSelect(this.selectedItem)
            }
            this.close()
            return
        }
        this.scrollToSelectedItem()
      }
    },

    scrollToSelectedItem () {
      const optionNodes = this.$refs.dropdown.querySelectorAll('.ui-items__option')
      const selectedNode = optionNodes[this.selectedIndex]

      if (selectedNode) {
        VueScrollTo.scrollTo(selectedNode, -1, {
          container: this.$refs.dropdown,
          offset: -100
        })
      }
    },

    hasSlot (slot) {
      return !!this.$slots[slot]
    },

    findItemByVal (value, items) {
      if (!value || !items || items.length === 0) return null

      let found = null

      for (let i in items) {
        if (items[i].children) {
          found = this.findItemByVal(value, items[i].children)
        }

        if (found) return found

        if (items[i] === value || items[i].value === value) return items[i]
      }

      return found
    },

    handleSelect (item) {
      if (this.isDisabled) return
      if (this.hasFilter) this.filter = ''

      /**
       * Emits when model changes with .value or raw string.
       * @event input
       * @type String | Number
       */

      this.$emit('input', item ? item.value || item : this.nullOptionValue)

      this.close()
    },

    handleNestedToggle (item) {
      item.expand = !item.expand
    }
  }
}
</script>

<style lang="scss" scoped>
@import "../../styles/all";

.ui-select {
  @include ui-wrap-soft(block);

  &__wrap {
    @include ui-wrap(flex);

    flex-wrap: wrap;
  }

  &__dock {
    &_left {
      margin-left: -0.5em;
    }

    &_caret {
      position: absolute;
      top: 1px;
      right: 0;
      bottom: 1px;
      text-align: right;
      width: 200px; //
      line-height: inherit;
      display: flex;
      align-items: center;
      justify-content: center;

      &_open {
        transform: rotateX(180deg);
      }
    }
  }

  &__label {
    display: block;
    width: 100%;
    clear: both;
    line-height: 5; //
    padding-top: 0.5em;
    padding-bottom: 0.5em;

    &_is-error {
      color: red; //
    }
  }

  &__button {
    width: 100%;

    /deep/ .ui-button__button {
      @include ui-can-error();

      text-align: left;
      padding-right: 10 * 0.8; //

      &[fill="none"] {
        border-radius: 0;
        border-width: 0 0 1px 0;
        border-color: grey; //
        border-style: solid;
        padding: 0 10px 0 0; //
        height: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      &[fill="frame"] {
        .ui-select__button-lbl_placeholder {
          color: transparentize(white, 0.75); //
        }
      }
    }
  }

  &__button-icon {
    height: 5; //
    border: 1px solid black; //
    margin-right: 0.4em;

    @include respond-below(xs) {
      display: none;
    }
  }

  &__button-lbl {
    &_placeholder-required::after {
      content: "*";
      display: inline;
      color: orange;
      margin-left: -1ch * 0.3;
    }
  }

  &__input {
    margin: 0 0.75em 0.75em 0.75em;

    /deep/ .ui-text__input {
      background-color: #edeef1 !important;
      border-color: #edeef1 !important;
    }

    &_sticky {
      position: sticky;
      top: 1em;
      padding-bottom: 1em;
    }
  }

  &__dock_left ~ &__input {
    padding-left: 10 * 1.1;
  }

  // DROPDOWN

  &__dropdown {
    @include ui-base(block);
    @include scrollbar-awesome();

    flex-wrap: wrap;
    position: absolute;
    top: 10; //
    min-width: 100%;
    max-height: 45vh;
    overflow-x: hidden;
    overflow-y: auto;
    z-index: 10; //
    margin-top: 3px;
    transition: none;
    display: none;
    box-shadow: black; //
    border-radius: 10px; //
    padding: 1em 0;
    max-width: 100%;
    background: white; //

    /*
      todo: this is not consistent behaviour, need to get rid of it
      but we'll keep it to save capability
    */
    &[fill="none"],
    &[look="none"] {
      background: white !important; //
    }

    &_occupies-space {
      position: relative;
      top: auto;
      left: auto;
      right: auto;

      @include respond-below(sm) {
        max-height: unset;
      }
    }

    &_sticky-filter {
      overflow: hidden;
      padding: 0;
    }
  }

  /*
      todo: this is not consistent behaviour, need to get rid of it
      but we'll keep it to save capability
  */
  &_is-open /deep/ .ui-button__button[fill="solid"] {
    -moz-border-radius: 10px 10px 0 0; //
    border-radius: 10px 10px 0 0; //
  }

  &_is-open &__dropdown {
    transition: all ease-out 200ms;
    display: block;
  }

  &__options {
    &_sticky-filter {
      overflow: auto;
      max-height: 36vh;
      padding-bottom: 1em;

      @include scrollbar-awesome();
    }
  }

  &__root-item {
    color: grey; //
    margin: 0 1em;
    padding: 0.4em 0.75em;
    display: block;
    line-height: 10 * 0.5; //
    white-space: nowrap;
    cursor: pointer;
    position: relative;
    border-radius: 3px;

    /deep/ .ui-icon {
      color: black; //
      vertical-align: middle;
    }

    &:focus,
    &:active {
      outline: none;
    }

    &:hover {
      outline: none;
      color: black; //
      background-color: var(--ui-col-secondary);
      opacity: 0.8;

      /deep/ .ui-icon {
        color: black; //
      }
    }

    &:last-child {
      display: none;
    }

    &:last-child:first-child {
      display: block;
    }

    &_active {
      outline: none;
      color: black; //
      background-color: var(--ui-col-primary);
      opacity: 0.8;

      &:hover {
        background-color: var(--ui-col-primary);
        color: var(--ui-col-secondary-contrast);
      }
    }

    &-empty {
      color: red;
    }
  }
}
</style>
