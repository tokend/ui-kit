Docks on the sides available if you want to add some icons. Slot is squared and automatically add margin to the input.
Don't try to put buttons there.

```jsx
<ui-text is-error>
  <span slot="dock-left" style="background: blue; color: white; padding: 2px">
    (L)
  </span>
  <span slot="dock-right" style="background: red; color: white; padding: 2px">
    (R)
  </span>
</ui-text>
```

```jsx
<ui-text>
  <span slot="dock-right" style="background: red; color: white; padding: 2px">
    (R)
  </span>
</ui-text>
```

```jsx
<ui-text>
  <span slot="dock-left" style="background: blue; color: white; padding: 2px">
    (L)
  </span>
</ui-text>
```

Very basic inputs with flags.

```jsx

<ui-text look="secondary">
  Lable is here
</ui-text>

<ui-text is-required placeholder="This is required">
    Normal input, no errors.
    <span slot="error">You should not see it.</span>
</ui-text>


<ui-text is-error  placeholder="Contains errors" class="ui-text__error">
    Input example with an error
    <div slot="error">Fok, Something went wrong</div>
</ui-text>


<ui-text is-disabled placeholder="Placeholder">
    You can't edit this.
    <span slot="error">You should not see it.</span>
</ui-text>


```

Inputs can looks differently like everything else. Avoid using this feature w/o necessity.

```jsx
<ui-text look="primary">
    Normal input, no errors.
    <span slot="error">You should not see it.</span>
</ui-text>

<ui-text fill="solid">
    Normal input, no errors.
    <span slot="error">You should not see it.</span>
</ui-text>

<ui-text fill="solid" is-disabled value="kjhagsdhj">
    Normal input, no errors. Disabled.
    <span slot="error">You should not see it.</span>
</ui-text>


```

For number inputs there may be added boundary and step props.

```jsx
<ui-text type="number" min="0" step="1" max="100">
    Only numeric values from 0 to 100 allowed.
</ui-text>

<ui-text type="number" min="-100" step="0.01" max="10">
    Allowed decimal values from -100 to 10 with 2 digits after point.
</ui-text>
```
