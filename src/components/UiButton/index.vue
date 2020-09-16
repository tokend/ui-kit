<script>
export default {
  name: 'UiButton',
  functional: true,

  props: {
    /** Set basic appearance of the element.<br>
     * @value: < primary | secondary | alt  >
     */
    look: { type: String, default: 'primary' },
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

<style lang='scss'>
@import '../../styles/all';

.ui-button {
  @include ui-wrap(inline-block);
  --ui-button-color: white;
  --ui-button-background-primary-color: var(--ui-col-primary);
  --ui-button-background-secondary-color: var(--ui-col-secondary);
  --ui-button-border: none;
  --ui-button-hover-cursor: pointer;
  --ui-button-active-transition: none;
  --ui-button-active-transform: none;
  --ui-button-text-align: center;
  --ui-button-padding: 0;
  --ui-button-font-size: var(--ui-font-size);
  --ui-button-width: 20rem;
  --ui-button-font-weight: 700;
  --ui-button-border-radius: var(--ui-border-radius);
  --ui-button-box-shadow: var(--ui-box-shadow);
  --ui-button-line-height: var(--ui-line-height);

  position: unset;

  &__button {
    @include ui-base(block);
    @include ui-can-fill(var(--ui-button-background-primary-color),
     var(--ui-button-background-secondary-color), var(--ui-button-box-shadow));
    @include ui-can-disabled();
    @include ui-can-click(var(--ui-button-active-transition),
     var(--ui-button-active-transform));

    -webkit-user-select: none;
    user-select: none;

    display: inline-block;
    cursor: var(--ui-button-hover-cursor);
    padding: var(--ui-button-padding);
    font-size: var(--ui-button-font-size);
    font-weight: var(--ui-button-font-weight);
    min-height: var(--ui-button-min-height);
    border-radius: var(--ui-button-border-radius);
    box-shadow: var(--ui-button-box-shadow);
    color: var(--ui-button-color);
    width: var(--ui-button-width);
    border: var(--ui-button-border);
    line-height: var(--ui-button-line-height);
    text-align: var(--ui-button-text-align);

    &:active, &:focus {
      transform: var(--ui-button-active-transform);
    }
  }
}
</style>
