Input for downloading the photo.

```jsx
<ui-photo :min-width="100" :min-height="100" :max-size="32">
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

Input with error message.

```jsx
<ui-photo>
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
    <template slot="error">Some error message</template>
</ui-photo>
```

Input with error handler (size).

```jsx
function errorHandler(){
    alert(Error('File size is invalid'))
}

<ui-photo @err-invalid-file-size="errorHandler" :max-size="0">
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
</ui-photo>
```

Input with error handler (type).

```jsx
function errorHandler(){
    alert(Error('File type is invalid'))
}

<ui-photo @err-invalid-file-type="errorHandler">
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
</ui-photo>
```

Input with error handler (dimensions).

```jsx
function errorHandler(){
    alert(Error('File dimensions is invalid'))
}

<ui-photo @err-invalid-file-dimensions="errorHandler" :min-width="1000" :min-height="1000">
    <template slot="mainLabel">Logo (Optional)</template>
    <template slot="saveDragedPhoto">Save</template>
    <template slot="cancelSavingDragedPhoto">Cancel</template>
</ui-photo>
```