Input for downloading the photo.

```jsx
<ui-photo>
    <input type="file" />   
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
    <template slot="mainLabel">Logo (Optional)</template>

</ui-photo>

```

Input with error.

```jsx
<ui-photo>
    <template slot="error">Some error message</template>
</ui-photo>

```