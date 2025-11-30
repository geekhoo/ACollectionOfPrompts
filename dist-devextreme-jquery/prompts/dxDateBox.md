# dxDateBox
This prompt equips a coding agent with the steps to instantiate the **dxDateBox** widget using the bundled assets.

## Setup steps
1. Link the base CSS bundle and pick a theme file:
   <link rel="stylesheet" href="../css/dx.common.css">
   <link rel="stylesheet" href="../css/dx.light.css">
2. Load jQuery before DevExtreme:
   <script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
3. Drop the core script that bundles every widget:
   <script src="../js/dx.all.js"></script>
4. Keep the **css/fonts** and **css/icons** folders next to these stylesheets so the icon/font paths remain correct.

## Starter snippet
The HTML below assumes the dist folder sits next to your page:
```html
<link rel="stylesheet" href="../css/dx.common.css">
<link rel="stylesheet" href="../css/dx.light.css">
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
<script src="../js/dx.all.js"></script>

<div id="dateBox"></div>

<script>
  $(function() {
    $('#dateBox').dxDateBox({
      width: '100%',
      height: 400,
      onInitialized: (e) => console.log('dxDateBox ready', e.component)
    });
  });
</script>
```

## Notes
- Swap `dx.light.css` for another theme (e.g., `dx.material.blue.light.css`) to change the look and feel.
- Pull in additional locales before `dx.all.js` from `../js/localization/<locale>.js` if you need translations.
- Replace the options object above with component-specific configuration (dataSource/items/value/etc.) when you build the final UI.
- These prompts assume the distilled assets live inside `dist-devextreme-jquery`; adjust the `../` paths if you move the folder.
