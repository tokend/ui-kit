```vue
<template>
    <div>
      <pre>Value: {{ simpleValue || "null" }}</pre>
      <p>
        <ui-select
          v-model="simpleValue"
          :options="simple"
          null-option="Any Item"
        >
          <span>Simple Dropdown with ability to deselect (value=null)</span>
        </ui-select>
      </p>
      
      <pre>Items:<br>{{simple}}</pre>
    </div>
</template>

<script>
export default {
  data() {
    return {
      simple: ["Item 1", "Item 2", "Item 3"],
      simpleValue: null,
    };
  }
};
</script>
```
