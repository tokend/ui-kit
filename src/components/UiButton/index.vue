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
  --ui-buttons-color: white;
  --ui-buttons-backgroundPrimary-color: rgb(128, 107, 252);
  --ui-buttons-backgroundSecondary-color: rgba(128, 107, 252, 0.7);
  --ui-buttons-border: none;
  --ui-buttons-text-align: center;
  --ui-buttons-hover-cursor: pointer;
  --ui-buttons-active-transition: none;
  --ui-buttons-active-transform: none;

  --ui-button-padding: 8px 16px;
  --ui-button-font-size: 14px;
  --ui-button-width: 200px;
  --ui-buton-font-weight: 700;
  --ui-button-min-height: 36px;
  --ui-button-border-radius: 2px;
  --ui-button-box-shadow: 0 0.5px 1px 0 rgba(0, 0, 0, 0.15);
  --ui-button-height: 36px;
  --ui-button-line-height: normal;

  --ui-button-disabledPrimary-background: rgb(128, 107, 252);
  --ui-button-disabledSecondary-background: rgba(128, 107, 252, 0.7);
  --ui-button-disabledPrimary-opacity: 0.5;
  --ui-button-disabledSecondary-opacity: 0.2;
  --ui-button-disabled-color: white;

  --ui-button-errorPrimary-background: rgb(216,26,37);
  --ui-button-errorSecondary-background: rgba(216,26,37,0.7);

  position: unset;

  &__button {
    @include ui-base(block);
    @include ui-can-fill(var(--ui-buttons-backgroundPrimary-color),var(--ui-buttons-backgroundSecondary-color), var(--ui-button-box-shadow));
    @include ui-can-disabled(var(--ui-button-disabledPrimary-background), var(--ui-button-disabledSecondary-background),var(--ui-button-disabledPrimary-opacity), var(--ui-button-disabledSecondary-opacity), var(--ui-button-disabled-color));
    @include ui-can-click(var(--ui-buttons-active-transition),var(--ui-buttons-active-transform));

    -webkit-user-select: none;
    user-select: none;

    display: inline-block;
    cursor: var(--ui-buttons-hover-cursor);
    width: 100%;

    padding: var(--ui-button-padding);
    font-size: var(--ui-button-font-size);
    font-weight: var(--ui-buton-font-weight);
    text-align: var(--ui-buttons-text-align);
    min-height: var(--ui-button-min-height);
    border-radius: var(--ui-button-border-radius);
    box-shadow: var(--ui-button-box-shadow);
    color: var(--ui-buttons-color);
    width: var(--ui-button-width);
    border: var(--ui-buttons-border);
    height: var(--ui-button-height);
    line-height: var(--ui-button-line-height);

    &:active, &:focus {
      transform: var(--ui-buttons-active-transform);
}
  }
}
</style>
