Generic country flag

```vue
<template>
  <div>
    <span v-for="c in countries" :key="c" class="flag-wrap">
      <ui-flag :country="c" />
    </span>
  </div>
</template>

<script>
export default {
  name: "UiFlagExample",
  data() {
    return {
      countries: ["ua", "gb", "ru", "us", "de", "ux"]
    };
  }
};
</script>
<style lang="scss" scoped>
.flag-wrap {
  display: inline-block;
  margin: 0.5em;
}
</style>
```
