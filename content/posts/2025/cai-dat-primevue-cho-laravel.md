---
title: "Cài đặt PrimeVue cho Laravel"
date: 2025-01-15T14:11:36+07:00
slug: /cai-dat-primevue-cho-laravel
description: Hướng dẫn setup PrimeVue UI cho Laravel 11.
image: images/PrimeVue.jpeg
caption: PrimeVue
categories:
  - laravel
  - web development
tags:
  - laravel
draft: false
---

### Download
PrimeVue is available for download on npm registry.

### Using npm
```
npm install primevue @primevue/themes
```

### Using yarn
```
yarn add primevue @primevue/themes
```

### Using pnpm
```
pnpm add primevue @primevue/themes
```

### Plugin
PrimeVue plugin is required to be installed as an application plugin to set up the default configuration. The plugin is lightweight, and only utilized for configuration purposes.
```
import { createApp } from 'vue';
import PrimeVue from 'primevue/config';

const app = createApp(App);
app.use(PrimeVue);
```

### Theme
Configure PrimeVue to use a theme like Aura.
```
import { createApp } from 'vue';
import PrimeVue from 'primevue/config';
import Aura from '@primevue/themes/aura';

const app = createApp(App);
app.use(PrimeVue, {
    theme: {
        preset: Aura
    }
});
```

### Verify
Verify your setup by adding a component such as Button. Each component can be imported and registered individually so that you only include what you use for bundle optimization. Import path is available in the documentation of the corresponding component.
```
import Button from "primevue/button"

const app = createApp(App);
app.component('Button', Button);
```
