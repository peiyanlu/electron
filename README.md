# @peiyanlu/electron

一个分层设计的 Electron 工具集，提供 backend、preload 和 frontend 的模块化能力封装。

## 后端

```ts
import {
  checkSquirrel,
  ElectronHost,
  getIconExt,
  IpcHost,
  isDev,
  onChildWindowOpenUrl,
  showAndFocus,
} from '@peiyanlu/electron/backend'
import { Menu } from 'electron'
import { join } from 'path'
import { ElectronSvgHandler } from './electron/IpcHandler'


const file = join(__dirname, '..', `renderer/${ MAIN_WINDOW_VITE_NAME }/index.html`)
const frontendURL = MAIN_WINDOW_VITE_DEV_SERVER_URL ?? file


const getIcon = (root: string, tray?: boolean) => join(root, 'icons', `icon.${ getIconExt(tray) }`)

const appIcon = getIcon(__dirname)
const trayIcon = getIcon(isDev ? __dirname : process.resourcesPath, true)


if (checkSquirrel()) {
  ElectronHost.shutdown()
}

ElectronHost.startup({
  ipcHandlers: [ ElectronSvgHandler ],
})

ElectronHost.openMainWindow({
    webPreferences: {
      preload: require.resolve('./preload.js'),
      sandbox: true,
    },
    width: 1200,
    height: 750,
    icon: appIcon,
    frontendURL,
    hideAppMenu: true,
    singleInstance: true,
    devTools: true,
    backgroundColor: '#141218',
    beforeReady: () => {
      onChildWindowOpenUrl()
    },
    tray: {
      icon: trayIcon,
      menu: Menu.buildFromTemplate([
        {
          label: '打开',
          click: () => {
            showAndFocus(ElectronHost.mainWindow)
          },
        },
        {
          label: '退出',
          click: () => {
            ElectronHost.shutdown()
          },
        },
      ]),
      title: `${ APP_NAME } ${ APP_VERSION }`,
    },
  })
  .then(async (window) => {
    if (!window) return
    //   DO SOMETHING
  })

IpcHost.addListener('changeTheme', (_e, data: string) => {
  console.log('changeTheme:receiver', data)
})
```

## Preload

```ts
import { contextBridge, ipcRenderer } from 'electron'
import { frontendApiKey, getFrontendApi } from '@peiyanlu/electron/preload'


contextBridge.exposeInMainWorld(frontendApiKey, getFrontendApi(ipcRenderer))
```

## 前端

```ts
import { svgoChannel, SvgoIpcInterface } from '@/electron/IpcInterface'
import { ElectronApp, IpcApp } from '@peiyanlu/electron/frontend'


ElectronApp.startup()
const svgoIpc = IpcApp.makeIpcFunctionProxy<SvgoIpcInterface>(svgoChannel, 'callMethod')


IpcApp.send('changeTheme', 'light')
```
