Input for downloading the photo.

```jsx
<ui-photo v-bind:min-width="100" v-bind:min-height="100" v-bind:max-size="32">
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
</ui-photo>
```

Disabled input.

```jsx
<ui-photo disabled>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
</ui-photo>
```

Input with error message and available input.

```jsx
<ui-photo>
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
    <template slot="error">Some error message</template>
</ui-photo>
```

Disabled error input.

```jsx
<ui-photo error-disabled>
    <template slot="error">Some error message</template>
</ui-photo>
```


