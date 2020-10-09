<template>
  <div
    class="ui-items"
    v-if="filteredOptions.length > 0">
    <div
      v-for="(item, i) in filteredOptions"
      :key="i"
      class="ui-items__option"
    >
      <a
        class="ui-items__list-item"
        :class="{
          'ui-items__list-item_active': item.value === currentValue,
          'ui-items__list-item_is-parent': !!item.children,
          'ui-items__list-item_selected': item.value === selectedValue
        }"
        tabindex="0"
        @click.stop="handleItemClick(item)"
      >
        <img
          v-if="item.icon"
          :src="item.icon"
          class="ui-items__list-item-icon"
        >
        <span class="ui-items__list-item-txt">
          {{ item.label }}
        </span>
        <a
          class="ui-items__list-item__caret"
          :class="{ 'ui-items__list-item__caret_open': item.expand }"
          v-if="!!item.children"
          @click.stop="$emit('nested-toggle', item)"
        >
          <ui-icon icon="arrow-right" />
        </a>
      </a>
      <ui-select-options
        class="ui-items__sub"
        v-if="item.children && item.expand"
        :current-value="currentValue"
        :selected-value="selectedValue"
        :items="item.children"
        @option-select="$emit('option-select', $event)"
        @nested-toggle="$emit('nested-toggle', $event)"
        :allow-select-group="allowSelectGroup"
      />
    </div>
  </div>
</template>

<script>

export default {
  props: {
    items: { type: Array, required: true },
    currentValue: { type: [Number, String, Object], default: null },
    selectedValue: { type: [Number, String, Object], default: null },
    /** Allow user to select select an item with child nodes */
    allowSelectGroup: { type: Boolean, default: false },
  },
  computed: {
    filteredOptions () {
      return this.items.filter(option => option.show && !option.hidden)
    }
  },
  methods: {
    handleItemClick (item) {
      !this.allowSelectGroup && item.children
        ? this.$emit('nested-toggle', item)
        : this.$emit('option-select', item)
    }
  }
}
</script>

<style lang="scss" scoped>
@import "../../styles/all";

.ui-items {
  user-select: none;
  display: block;
  clear: both;

  &__list-item {
    color: grey; //
    margin: 0 1em 0 1em;
    padding: 0.3em 0.75em;
    display: flex;
    align-items: center;
    line-height: 1.5;
    white-space: nowrap;
    cursor: pointer;
    position: relative;
    border-radius: 3px;

    &:hover {
      color: black;
      background-color: var(--ui-col-secondary); //
      opacity: 0.8;
    }

    &:focus,
    &:active {
      outline: none;
    }

    &_is-parent {
      padding-right: 10px; //
    }

    &_active {
      outline: none;
      color: var(--ui-col-primary-contrast);
      background-color: var(--ui-col-primary); //
      opacity: 0.8;
      font-weight: bold;

      &:hover {
        background-color: var(--ui-col-primary); //
        color: var(--ui-col-primary-contrast);
      }
    }

    &_selected {
      color: black;
      background-color: var(--ui-col-secondary); //
      opacity: 0.8;
    }

    &-icon {
      width: 10 / 1.5; //
      margin-right: 0.4rem;
    }

    &__caret {
      position: absolute;
      right: 0;
      top: 0;
      bottom: 0;
      display: flex;
      width: 200px; //
      align-items: center;
      text-align: center;
      justify-content: center;
      transform-origin: 50%;
      font-size: 0.8em;
      border-left: 1px solid transparent;

      &_open {
        /deep/ .ui-icon {
          transform: rotateZ(90deg);
        }
      }
    }
  }

  &:hover &__caret {
    border-left: 1px solid white; //
  }

  &__sub {
    /deep/ .ui-items__list-item {
      margin-left: 2.25em;
    }
  }
}
</style>
