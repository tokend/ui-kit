Phone Input Support all features that input text support

Default:
```jsx
<ui-phone value="+380998765678">
  Please provide your phone
</ui-phone>
```

With only pinned countries.
```jsx
<ui-phone filter-countries>
  Please provide your phone
</ui-phone>
```

Disabled:
```jsx
<ui-phone value="+380998765678" is-disabled>
  This field is not allowed to fill
</ui-phone>
```

Required:
```jsx
<ui-phone value="+380998765678" is-required>
  This field is required
</ui-phone>
```


With error:
```jsx
<ui-phone value="+380998765678" is-error>
  Please provide your phone

  <span slot="error">The value of the field is invalid</span>
</ui-phone>
```

