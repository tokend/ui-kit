A simple field:

```jsx
<ui-text is-required>
  Provide a first name
</ui-text>
```

When we got an error

```jsx
<ui-text is-error is-required>
  First Name 
  <template slot="error">This field is required</template>
</ui-text>

<br>

<ui-text is-error value="John Doe">
  Provide a first name
  <template slot="error">The value of the field is invalid</template>
</ui-text>
```

When disabled

```jsx
<ui-text is-disabled>First name</ui-text>
<br>
<ui-text is-disabled value="Jack Smith">First name</ui-text>
```
