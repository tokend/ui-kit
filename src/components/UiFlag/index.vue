<script>
export default {
  name: 'UiFlag',
  functional: true,

  props: {
    country: { type: String, default: 'none' }
  },

  render (createElement, { data, props }) {
    return createElement(
      'div',
      {
        ...data,
        class: [data.class, 'ui-flag', `ui-flag_${props.country}`]
      },
      props.country
    )
  }
}
</script>

<style lang="scss">
@import "../../styles/all";
@import "../../styles/flags";

:root {
  --ui-col-flag-background: #9498a6;
  --ui-col-flag-color: var(--ui-col-light);
}

.ui-flag {
  background-position: $ui-flag-height $ui-flag-width;
  height: $ui-flag-height;
  width: $ui-flag-width;

  background-image: url("../../assets/flags.png");
  background-repeat: no-repeat;

  // text fallback
  display: inline-flex;
  background-color: var(--ui-col-flag-background);
  color: var(--ui-col-flag-color);
  justify-content: center;
  align-items: center;
  font-size: $ui-flag-height * 0.75;
  line-height: $ui-flag-height;
  overflow: hidden;

  @each $flag, $bg-pos in $flags {
    &.ui-flag_#{$flag} {
      background-position: $bg-pos;
      color: transparent;
    }
  }
}
</style>
