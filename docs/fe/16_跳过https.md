

Android Chrom地址栏输入

```sh
chrome://flags/#unsafely-treat-insecure-origin-as-secure
```

vite.config.ts 开启server.host为true,将监听所有地址，包括局域网和公网地址。

```
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
  server: {
    host: true,
    open: true
  }
})

```

