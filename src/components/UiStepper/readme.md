Stepper for filling information.

```jsx
<ui-stepper
    :titles="[
        'Title 1',
        'Title 2',
        'Title 3',
      ]">
     <template #0="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            @click="next"
            look="primary"
            fill="solid">
            Continue
          </ui-button>
        </div>
      </template>
      <template #1="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            @click="next"
            look="primary"
            fill="solid">
            Continue
          </ui-button>
        </div>
      </template>
      <template #2="{ next }">
        <div class="container">
          <h2 class="container-title">
            Content
          </h2>
          <ui-button
            look="primary"
            fill="solid">
            Submit
          </ui-button>
        </div>
      </template>
</ui-stepper>

```
