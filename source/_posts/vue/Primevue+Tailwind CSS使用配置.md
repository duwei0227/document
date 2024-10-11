---
layout: post
title: Primevue+Tailwind CSS使用配置
categories: [Vue]
permalink: vue/primevue_tailwindcss.html
---


#### 1、创建vue项目

```shell
npm create vue@latest
```



#### 2、安装primevue

```shell
npm install primevue
```



在`main.js`中启用`primevue`

```javascript
import PrimeVue from 'primevue/config';
const app = createApp(App);

app.use(PrimeVue, { /* options */ });
```



#### 3、启用组件自动导入

```shell
npm i unplugin-vue-components -D
npm i @primevue/auto-import-resolver -D
```



在`vite.config.js`中配置组件自动解析

```javascript

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite';
import {PrimeVueResolver} from '@primevue/auto-import-resolver';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    Components({
      resolvers: [
        PrimeVueResolver()
      ]
    }),
  ]
})

```



#### 4、安装主题theme

```shell
npm install @primevue/themes
```

在`main.js`文件中配置主题：

```javascript

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

当前`primevue`内置支持4种主题：`Aura, Material, Lara, Nora `



#### 5、使用 primevueicons

```shell
npm install primeicons
```



#### 6、集成 tailwind css

```shell
npm i tailwindcss-primeui

-- 生成 tailwind.config.js文件
npx tailwindcss init
```

在应用`sytle.css`中加入以下内容：

```javascript
@tailwind base;
@tailwind components;
@tailwind utilities;
```



在 `tailwind.config.js`文件中引入`primevue`支持

```javascript
const primeui = require('tailwindcss-primeui');

export default {
    content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
    darkMode: ['selector', '[class="p-dark"]'],
    plugins: [primeui]
};
```



#### 7、Noir主题js内容

基于 `Aura`定制主题

```javascript
import { definePreset } from '@primevue/themes';
        import Aura from '@primevue/themes/aura';

        const Noir = definePreset(Aura, {
            semantic: {
                primary: {
                50: '{surface.50}',
                100: '{surface.100}',
                200: '{surface.200}',
                300: '{surface.300}',
                400: '{surface.400}',
                500: '{surface.500}',
                600: '{surface.600}',
                700: '{surface.700}',
                800: '{surface.800}',
                900: '{surface.900}',
                950: '{surface.950}'
                },
                colorScheme: {
                    light: {
                        primary: {
                        color: '{primary.950}',
                        contrastColor: '#ffffff',
                        hoverColor: '{primary.900}',
                        activeColor: '{primary.800}'
                        },
                        highlight: {
                        background: '{primary.950}',
                        focusBackground: '{primary.700}',
                        color: '#ffffff',
                        focusColor: '#ffffff'
                        },
                    },
                    dark: {
                        primary: {
                        color: '{primary.50}',
                        contrastColor: '{primary.950}',
                        hoverColor: '{primary.100}',
                        activeColor: '{primary.200}'
                        },
                        highlight: {
                        background: '{primary.50}',
                        focusBackground: '{primary.300}',
                        color: '{primary.950}',
                        focusColor: '{primary.950}'
                        }
                    }
                }
            }
        });

        export default Noir;
        
```

