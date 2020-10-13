Input for test

```jsx
<ui-photo>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
    <template slot="error">Some error message</template>
</ui-photo>
```

Input for downloading the photo.

```jsx
<ui-photo>
    <input type="file" />   
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
    <template slot="mainLabel">Logo (Optional)</template>
</ui-photo>
```

Input with error-only.

```jsx
<ui-photo error-message="Some error" error-disabled>
    <template slot="error">Some error message</template>
</ui-photo>
```