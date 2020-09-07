<script>
export default {
  name: 'UiButton',
  functional: true,

  props: {
    /** Set basic appearance of the element.<br>
     * @value: < default | primary | secondary | alt  >
     */
    look: { type: String, default: 'default' },
    /** Set basic fill of the element.<br>
     * @value: < solid | frame | frame-fill | none >
     */
    fill: { type: String, default: 'solid' },

    /**
     * Button type attribute
     */
    type: { type: String, default: 'button' },
    isRequired: { type: Boolean, default: false },
    isError: { type: Boolean, default: false },
    isDisabled: { type: Boolean, default: false }
  },

  render (createElement, { data, props, slots }) {
    const { attrs, on, ...otherData } = data
    const buttonElement = createElement(
      'button',
      {
        class: 'ui-button__button',
        on,
        attrs: {
          ...attrs,
          look: props.look,
          fill: props.fill,
          type: props.type,
          'is-required': props.isRequired,
          'is-error': props.isError,
          'is-disabled': props.isDisabled,
          disabled: props.disabled || props.isDisabled
        }
      },
      slots().default
    )

    return createElement(
      'div',
      { ...otherData, class: [otherData.class, 'ui-button'] },
      [buttonElement]
    )
  }
}
</script>

<style lang="scss">
@import "../../styles/all";

:root {
  --ui-button-padding: 0.25rem 1.75rem;
}

.ui-button {
  @include ui-wrap(inline-block);

  position: unset;

  &__button {
    @include ui-base(block);
    @include ui-can-fill();
    @include ui-can-disabled();
    @include ui-can-click();

    -webkit-user-select: none;
    user-select: none;

    display: inline-block;
    cursor: pointer;
    width: 100%;

    padding: var(--ui-button-padding);
    line-height: var(--ui-line-height) / 2;
    border-radius: var(--ui-border-radius);
  }
}
</style>
