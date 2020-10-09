```vue
<template>
  <div class="selects">
    <div class="select">
      <pre>Value: {{ simpleValue || "null" }}</pre>
      <p>
        <ui-select
          v-model="simpleValue"
          :options="simple"
          null-option="Any Item"
        >
          <span class="span-title">Simple Dropdown</span>
        </ui-select>
        <pre class="per-items">Items:<br>{{simple}}</pre>
      </p>
      <p>
        <ui-select
          v-model="simpleValue"
          is-error has-filter 
          :options="simple">
          Is Error
        </ui-select>
        <ui-select
          v-model="simpleValue"
          is-required
          has-filter
          :options="simple"
        >
          Is Required
        </ui-select>
        <ui-select
          v-model="simpleValue"
          is-disabled
          has-filter
          :options="simple"
        >
          Is Disabled
        </ui-select>
      </p>

      <pre class="per-items">Items:<br>{{simple}}</pre>
    </div>
    <div class="select">
      <pre>Value: {{ complexValue || "null" }}</pre>

      <p>
        <ui-select v-model="complexValue" has-filter :options="complex">
          <span class="span-title">Complex List</span>
        </ui-select>
      </p>

      <pre class="per-items">Items:<br>{{complex}}</pre>
    </div>
    <div class="select">
      <pre>Value: {{ nestedValue || "null" }}</pre>
      <p>
        <ui-select
          v-model="nestedValue"
          has-filter
          :options="nested"
          placeholder="Select Placeholder Text"
          search-placeholder="Quick search"
          null-option="Any car"
        >
          <span class="span-title">Nested List</span>
        </ui-select>
      </p>
      <p>
        Notice: Any list option can be a string, object with label or value, or
        both.
      </p>
      <pre class="per-items">Items:<br>{{nested}}</pre>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      simple: ["Item 1", "Item 2", "Item 3"],
      simpleValue: null,
      complex: [
        { value: 1, label: "First Item" },
        { value: 2, label: "Second Item" }
      ],
      complexValue: null,
      nested: [
        { label: "Apple", value: "APPLE" },
        { label: "Pork", value: "PORK" },
        { label: "Cucumber" },
        {
          label: "Cars",
          value: "ALL_CARS_GROUP",
          children: [
            { label: "Nice Car", value: "nice" },
            { label: "Broken Car" },
            "Generic Car"
          ]
        }
      ],
      nestedValue: "ALL_CARS_GROUP"
    };
  }
};
</script>

<style lang="scss" scoped>
@import "../../styles/all";

  .selects {
    display: flex;
    width: 100%;
  }
  .select {
      width: calc(100% / 3);
      margin: 0 15px;
    }
  .span-title {
    font-size: 1.2rem;
    background-color: white;
    padding: 0 0.5rem;
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    line-height: 1;
    margin: -0.5rem 1rem;
    color: var(--ui-col-primary);
  }
  .per-items {
    font-size: 13px;
  }
  p > div {
    margin: 30px 0;
  }
</style>
```


