# svelte-wheel-picker

A customizable and interactive wheel picker component for Svelte applications, inspired by the classic iOS time selector.

[![npm version](https://badge.fury.io/js/svelte-wheel-picker.svg)](https://www.npmjs.com/package/svelte-wheel-picker)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Features

- Customizable wheel picker with smooth scrolling and inertia
- Configurable number of visible options and density
- Perspective effect for a 3D wheel appearance
- Hover effects with unwrapping and stamping effect options
- Responsive design with options to fill parent container
- Touch and mouse event support for mobile and desktop
- No dependencies

## Installation

```bash
npm install svelte-wheel-picker
```

## Basic Usage

```svelte
<script>
  import WheelPicker from 'svelte-wheel-picker';

  let data = [
    {value: 1, label: "Option 1"},
    {value: 2, label: "Option 2"},
    {value: 3, label: "Option 3"},
    // Add more options as needed
  ];
</script>

<WheelPicker {data} />
```

## Advanced Usage

### Customizing Appearance and Behavior

```svelte
<script>
  import WheelPicker from 'svelte-wheel-picker';

  let data = [
    // Your data array here
  ];

  let options = {
    visibleOptions: 5,
    density: 10,
    perspective: 30,
    friction: 3,
  };

  let rollOnHover = {
    enable: true,
    unwrap: 85,
    stamp: true,
    fixList: false,
    transformSpeed: 0.3
  };

  let fillParent = {
    enable: true
  };
</script>

<WheelPicker 
  {data}
  {options}
  {rollOnHover}
  {fillParent}
  classes="custom-class"
  style="background-color: #f0f0f0;"
/>
```

## Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `data` | `Array` | `[...]` | Array of objects with `value` and `label` properties |
| `selectedOption` | `object` | `data[0]` | Initially selected option |
| `classes` | `string` | `""` | Custom CSS classes |
| `style` | `string` | `""` | Custom inline styles |
| `options` | `object` | `{}` | Configuration options for the wheel |
| `rollOnHover` | `object` | `{enable: false}` | Configuration for hover effects |
| `fillParent` | `object` | `{enable: false}` | Configuration for filling parent container |

### `options` Object Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `visibleOptions` | `number` | `7` | Number of visible options |
| `density` | `number` | `visibleOptions * 2` | Density of options (affects scaling) |
| `marginY` | `number` | `2` | Vertical margin (in %) |
| `perspective` | `number` | `0` | Perspective effect (0-180) |
| `staticPointerTimer` | `number` | `200` | Timer for static pointer (in ms) |
| `friction` | `number` | `4` | Friction for inertia effect |
| `overflow` | `string` | `"hidden"` | Overflow behavior ("hidden" or "visible") |

### `rollOnHover` Object Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enable` | `boolean` | `false` | Enable hover effects |
| `unwrap` | `number` | `85` | Unwrap percentage on hover (0-99) |
| `stamp` | `boolean` | `false` | Enable stamping effect |
| `fixList` | `boolean` | `true` | Fix list position on hover |
| `transformSpeed` | `number` | `0.3` | Animation speed for transformations |

### `fillParent` Object Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `enable` | `boolean` | `false` | Enable filling parent container |
| `height` | `string` | `undefined` | Custom height when not filling parent |

## Events

The WheelPicker component does not emit custom events, but it updates the `selectedOption` internally. You can bind to this value to react to changes:

```svelte
<script lang="ts">
  import WheelPicker from 'svelte-wheel-picker';
  import type {DataOption} from 'svelte-wheel-picker'
  
  let selectedOption:DataOption;
</script>

<WheelPicker bind:selectedOption {...otherProps} />

{#if selectedOption}
  <p>Selected: {selectedOption.label} (Value: {selectedOption.value})</p>
{/if}
```

## Styling

The WheelPicker component uses inline styles for most of its styling needs. You can customize its appearance using the `classes` and `style` props, as well as by adjusting the `options` object.

## Browser Compatibility

This component uses modern JavaScript features and CSS properties. Ensure you're using it in a compatible environment or provide appropriate polyfills.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License.

## About

Created by [um-dev](https://urmoov.dev)