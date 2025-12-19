# Tailwind CSS

Tailwind CSS is a utility-first CSS framework for rapidly building custom user interfaces.

## Installation

```bash
# Via npm
npm install -D tailwindcss
npx tailwindcss init

# Via CDN (for development only)
<script src="https://cdn.tailwindcss.com"></script>
```

## Configuration

```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}",
    "./public/index.html"
  ],
  theme: {
    extend: {
      colors: {
        primary: '#3B82F6',
        secondary: '#10B981',
      },
      spacing: {
        '128': '32rem',
      },
    },
  },
  plugins: [],
}
```

```css
/* styles.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Layout

### Container

```html
<div class="container mx-auto">
  <!-- Content centered with max-width -->
</div>
```

### Display

```html
<div class="block">Block</div>
<div class="inline-block">Inline Block</div>
<div class="inline">Inline</div>
<div class="flex">Flexbox</div>
<div class="grid">Grid</div>
<div class="hidden">Hidden</div>
```

### Flexbox

```html
<!-- Flex container -->
<div class="flex justify-center items-center">
  <div>Item 1</div>
  <div>Item 2</div>
</div>

<!-- Flex direction -->
<div class="flex flex-row">Horizontal</div>
<div class="flex flex-col">Vertical</div>
<div class="flex flex-row-reverse">Reverse horizontal</div>

<!-- Justify content -->
<div class="flex justify-start">Start</div>
<div class="flex justify-center">Center</div>
<div class="flex justify-end">End</div>
<div class="flex justify-between">Space between</div>
<div class="flex justify-around">Space around</div>

<!-- Align items -->
<div class="flex items-start">Start</div>
<div class="flex items-center">Center</div>
<div class="flex items-end">End</div>
<div class="flex items-stretch">Stretch</div>

<!-- Flex wrap -->
<div class="flex flex-wrap">Wrap</div>
<div class="flex flex-nowrap">No wrap</div>

<!-- Flex grow/shrink -->
<div class="flex">
  <div class="flex-1">Grow</div>
  <div class="flex-none">No grow</div>
</div>
```

### Grid

```html
<!-- Grid columns -->
<div class="grid grid-cols-3 gap-4">
  <div>1</div>
  <div>2</div>
  <div>3</div>
</div>

<!-- Responsive grid -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <!-- Content -->
</div>

<!-- Grid gap -->
<div class="grid grid-cols-3 gap-2">Gap 2</div>
<div class="grid grid-cols-3 gap-4">Gap 4</div>
<div class="grid grid-cols-3 gap-x-4 gap-y-2">Different gaps</div>
```

## Spacing

```html
<!-- Padding -->
<div class="p-4">Padding all sides</div>
<div class="px-4">Padding horizontal</div>
<div class="py-4">Padding vertical</div>
<div class="pt-4">Padding top</div>
<div class="pr-4">Padding right</div>
<div class="pb-4">Padding bottom</div>
<div class="pl-4">Padding left</div>

<!-- Margin -->
<div class="m-4">Margin all sides</div>
<div class="mx-auto">Margin horizontal auto (center)</div>
<div class="my-4">Margin vertical</div>
<div class="mt-4">Margin top</div>
<div class="-mt-4">Negative margin top</div>

<!-- Space between -->
<div class="flex space-x-4">
  <div>1</div>
  <div>2</div>
</div>
```

## Typography

```html
<!-- Font size -->
<p class="text-xs">Extra small</p>
<p class="text-sm">Small</p>
<p class="text-base">Base</p>
<p class="text-lg">Large</p>
<p class="text-xl">Extra large</p>
<p class="text-2xl">2xl</p>

<!-- Font weight -->
<p class="font-thin">Thin</p>
<p class="font-light">Light</p>
<p class="font-normal">Normal</p>
<p class="font-medium">Medium</p>
<p class="font-semibold">Semibold</p>
<p class="font-bold">Bold</p>

<!-- Text align -->
<p class="text-left">Left</p>
<p class="text-center">Center</p>
<p class="text-right">Right</p>
<p class="text-justify">Justify</p>

<!-- Text color -->
<p class="text-gray-500">Gray</p>
<p class="text-blue-600">Blue</p>
<p class="text-red-500">Red</p>

<!-- Text decoration -->
<p class="underline">Underline</p>
<p class="line-through">Line through</p>
<p class="no-underline">No underline</p>

<!-- Text transform -->
<p class="uppercase">UPPERCASE</p>
<p class="lowercase">lowercase</p>
<p class="capitalize">Capitalize</p>

<!-- Line height -->
<p class="leading-tight">Tight</p>
<p class="leading-normal">Normal</p>
<p class="leading-loose">Loose</p>
```

## Colors

```html
<!-- Background colors -->
<div class="bg-white">White</div>
<div class="bg-gray-100">Gray 100</div>
<div class="bg-blue-500">Blue 500</div>
<div class="bg-red-600">Red 600</div>

<!-- Text colors -->
<p class="text-gray-900">Dark gray text</p>
<p class="text-blue-600">Blue text</p>

<!-- Border colors -->
<div class="border border-gray-300">Border</div>
<div class="border-2 border-blue-500">Blue border</div>
```

## Borders

```html
<!-- Border width -->
<div class="border">Border all sides</div>
<div class="border-2">Border 2px</div>
<div class="border-t">Border top</div>
<div class="border-r">Border right</div>

<!-- Border radius -->
<div class="rounded">Rounded</div>
<div class="rounded-lg">Rounded large</div>
<div class="rounded-full">Fully rounded</div>
<div class="rounded-t">Rounded top</div>

<!-- Border style -->
<div class="border border-solid">Solid</div>
<div class="border border-dashed">Dashed</div>
<div class="border border-dotted">Dotted</div>
```

## Sizing

```html
<!-- Width -->
<div class="w-full">Full width</div>
<div class="w-1/2">Half width</div>
<div class="w-1/3">One third</div>
<div class="w-64">256px</div>
<div class="w-screen">Screen width</div>

<!-- Height -->
<div class="h-full">Full height</div>
<div class="h-screen">Screen height</div>
<div class="h-64">256px</div>

<!-- Max/Min width -->
<div class="max-w-sm">Max width small</div>
<div class="max-w-md">Max width medium</div>
<div class="max-w-lg">Max width large</div>
<div class="min-w-0">Min width 0</div>
```

## Positioning

```html
<!-- Position -->
<div class="relative">Relative</div>
<div class="absolute">Absolute</div>
<div class="fixed">Fixed</div>
<div class="sticky">Sticky</div>

<!-- Top/Right/Bottom/Left -->
<div class="absolute top-0 right-0">Top right corner</div>
<div class="absolute bottom-0 left-0">Bottom left corner</div>

<!-- Z-index -->
<div class="z-0">Z-index 0</div>
<div class="z-10">Z-index 10</div>
<div class="z-50">Z-index 50</div>
```

## Effects

```html
<!-- Shadow -->
<div class="shadow">Shadow</div>
<div class="shadow-md">Medium shadow</div>
<div class="shadow-lg">Large shadow</div>
<div class="shadow-none">No shadow</div>

<!-- Opacity -->
<div class="opacity-0">Invisible</div>
<div class="opacity-50">Half opacity</div>
<div class="opacity-100">Fully opaque</div>

<!-- Hover effects -->
<button class="hover:bg-blue-700">Hover me</button>
<a class="hover:underline">Hover link</a>
```

## Transitions

```html
<!-- Transition -->
<button class="transition duration-300 ease-in-out transform hover:scale-110">
  Hover to scale
</button>

<button class="transition-colors duration-200 bg-blue-500 hover:bg-blue-700">
  Color transition
</button>

<!-- Transform -->
<div class="transform scale-100 hover:scale-110">Scale on hover</div>
<div class="transform rotate-45">Rotated</div>
<div class="transform translate-x-4">Translated</div>
```

## Responsive Design

```html
<!-- Mobile first approach -->
<div class="w-full md:w-1/2 lg:w-1/3">
  <!-- Full width on mobile, half on tablet, third on desktop -->
</div>

<!-- Breakpoints -->
<div class="text-sm sm:text-base md:text-lg lg:text-xl">
  <!-- Font size increases with screen size -->
</div>

<!-- Hide on mobile, show on desktop -->
<div class="hidden lg:block">Desktop only</div>

<!-- Show on mobile, hide on desktop -->
<div class="block lg:hidden">Mobile only</div>
```

## Common Patterns

### Button

```html
<button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-700 transition">
  Click me
</button>

<button class="px-6 py-3 bg-green-500 text-white rounded-lg shadow-md hover:bg-green-600 transform hover:scale-105 transition">
  Fancy button
</button>
```

### Card

```html
<div class="max-w-sm rounded overflow-hidden shadow-lg">
  <img class="w-full" src="image.jpg" alt="Image">
  <div class="px-6 py-4">
    <div class="font-bold text-xl mb-2">Card Title</div>
    <p class="text-gray-700 text-base">
      Card description goes here.
    </p>
  </div>
  <div class="px-6 pt-4 pb-2">
    <span class="inline-block bg-gray-200 rounded-full px-3 py-1 text-sm font-semibold text-gray-700 mr-2">
      #tag1
    </span>
  </div>
</div>
```

### Navigation

```html
<nav class="bg-gray-800 p-4">
  <div class="container mx-auto flex justify-between items-center">
    <div class="text-white font-bold text-xl">Logo</div>
    <div class="flex space-x-4">
      <a href="#" class="text-gray-300 hover:text-white transition">Home</a>
      <a href="#" class="text-gray-300 hover:text-white transition">About</a>
      <a href="#" class="text-gray-300 hover:text-white transition">Contact</a>
    </div>
  </div>
</nav>
```

### Form

```html
<form class="max-w-md mx-auto p-6 bg-white rounded-lg shadow-md">
  <div class="mb-4">
    <label class="block text-gray-700 text-sm font-bold mb-2" for="email">
      Email
    </label>
    <input
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500"
      id="email"
      type="email"
      placeholder="Email"
    >
  </div>
  <div class="mb-6">
    <label class="block text-gray-700 text-sm font-bold mb-2" for="password">
      Password
    </label>
    <input
      class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:ring-2 focus:ring-blue-500"
      id="password"
      type="password"
      placeholder="Password"
    >
  </div>
  <button class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline w-full transition" type="submit">
    Sign In
  </button>
</form>
```

## Plugins

```bash
# Official plugins
npm install @tailwindcss/forms
npm install @tailwindcss/typography
npm install @tailwindcss/aspect-ratio
```

```javascript
// tailwind.config.js
module.exports = {
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

## Resources

- [Official Documentation](https://tailwindcss.com/docs)
- [Tailwind UI](https://tailwindui.com/)
- [Headless UI](https://headlessui.com/)
