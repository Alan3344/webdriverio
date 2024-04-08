---
id: electron
title: Electron
---

Electron 是一个使用 JavaScript、HTML 和 CSS 构建桌面应用程序的框架。通过将 Chromium 和 Node.js 嵌入到其二进制文件中，Electron 允许您维护一个 JavaScript 代码库并创建可在 Windows、macOS 和 Linux 上运行的跨平台应用程序 -无需本地开发经验。

WebdriverIO 提供了一项集成服务，可以简化与 Electron 应用程序的交互并使其测试变得非常简单。使用 WebdriverIO 测试 Electron 应用程序的优点是：

- ☺️ 自动下载正确的 Chromedriver 版本
- ⚒️ 访问Electron API，即：[`app`](https://www.electronjs.org/docs/latest/api/app)、[`browserWindow`](https://www.electronjs.org/docs/latest/api/browser-window), [`dialog`](https://www.electronjs.org/docs/latest/api/dialog) 和 [`mainProcess`](https://www.electronjs.org/docs/latest/api/process)。
- 🔄 Electron API 功能的自定义模拟
- 👤 能够定义自定义 API 处理程序来修改测试下的应用程序行为

您只需要几个简单的步骤即可开始。

## 入门

要启动新的 WebdriverIO 项目，请运行：

```sh
npm create wdio@latest ./
```

安装向导将指导您完成整个过程。当它询问您想要执行什么类型的测试时，请确保选择 _“Electron 应用程序的桌面测试”_。然后提供编译后的 Electron 应用程序的路径，例如 `./dist`，然后保留默认值或根据您的喜好进行修改。

配置向导将安装所有必需的包，并使用必要的配置创建 `wdio.conf.js` 或 `wdio.conf.ts` 来测试您的应用程序。如果您同意自动生成一些测试文件，您可以通过 `npm run wdio` 运行第一个测试。

就是这样🎉

## 配置

要使用该服务，您需要将 `electron` 添加到您的服务数组中并设置 Electron 功能，例如：

```js
// wdio.conf.js
import url from 'node:url';
import path from 'node:path';

const __dirname = url.fileURLToPath(new URL('.', import.meta.url));
export const config = {
  outputDir: 'logs',
  // ...
  services: ['electron'],
  capabilities: [{
      browserName: 'electron'
  }],
  // ...
};
```

**Note:** the `browserName` capability needs to be `electron` to have the service set up the automation session properly.

The service will attempt to find the path to your bundled Electron application if you use [Electron Forge](https://www.electronforge.io/) or [Electron Builder](https://www.electron.build/) as bundler. You can provide a custom path to the binary via custom service capabilities, e.g.:

```ts
capabilities: [{
  browserName: 'electron',
  'wdio:electronServiceOptions': {
    appBinaryPath: './path/to/bundled/electron/app.exe',
    appArgs: ['foo', 'bar=baz'],
  },
}],
```

## API Configuration

If you wish to use the electron APIs then you will need to import (or require) the preload and main scripts in your app. Somewhere near the top of your preload:

```ts
const isTest = process.env.NODE_ENV === 'test'
if (isTest) {
  require('wdio-electron-service/preload');
}
```

And somewhere near the top of your main index file (app entry point):

```ts
const isTest = process.env.NODE_ENV === 'test'
if (isTest) {
  require('wdio-electron-service/main');
}
```

The APIs should not work outside of WDIO but for security reasons it is encouraged to use dynamic imports wrapped in conditionals to ensure the APIs are only exposed when the app is being tested.

After importing the scripts the APIs should now be available in tests.

Currently available APIs:
- [`app`](https://www.electronjs.org/docs/latest/api/app)
- [`browserWindow`](https://www.electronjs.org/docs/latest/api/browser-window)
- [`dialog`](https://www.electronjs.org/docs/latest/api/dialog)
- [`mainProcess`](https://www.electronjs.org/docs/latest/api/process).

The service re-exports the WDIO browser object with the `.electron` namespace for API usage in your tests:

```ts
import { browser } from '@wdio/globals';

// in a test
const appName = await browser.electron.app('getName');
```

## Mocking Electron APIs

You can mock electron API functionality by calling the mock function with the API name, function name and mock return value. e.g. in a spec file:

```ts
await browser.electron.mock('dialog', 'showOpenDialog', 'dialog opened!');
const result = await browser.electron.dialog('showOpenDialog');
console.log(result); // 'dialog opened!'
```

## Custom Electron API

You can also implement a custom API if you wish. To do this you will need to define a handler in your main process:

```ts
import { ipcMain } from 'electron';

ipcMain.handle('wdio-electron', () => {
  // access some Electron or Node things on the main process
  return 'such api';
});
```

The custom API can then be called in a spec file:

```ts
const someValue = await browser.electron.api('wow');
console.log(someValue); // outputs: "such api"
```

## More Information

You can learn more about how to configure the [`wdio-electron-service`](https://www.npmjs.com/package/wdio-electron-service) in the [service docs](/docs/wdio-electron-service). Make sure to check out our [Electron boilerplate project](https://github.com/webdriverio/electron-boilerplate) that showcases how to intergate WebdriverIO in an example application.
