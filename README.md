# 일렉트론 배포서버를 깃헙이 아닌 로컬호스트로 설정하기

```js
{
  "name": "electron-app",
  "version": "1.0.0",
  "description": "My test electron app",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch": "nodemon --exec electron .",
    "start": "electron .",
    "dist": "rimraf dist && electron-builder"
  },
  "build": {
    "publish": [
      {
        "provider": "generic",
        "url": "http://localhost:3030/download"
      }
    ],
    "appId": "com.coderjeet.autoupdater",
    "productName": "Autoupdater app",
    "win": {
      "target": "nsis"
    },
    "directories": {
      "output": "dist"
    }
  },
  "author": "CoderJeet",
  "license": "ISC",
  "devDependencies": {
    "electron": "^19.0.9",
    "electron-builder": "^23.6.0",
    "nodemon": "^2.0.19"
  },
  "dependencies": {
    "electron-log": "^4.4.8",
    "electron-updater": "^5.3.0",
    "rimraf": "^5.0.0"
  }
}

```


```js
const { app, BrowserWindow, ipcMain, ipcRenderer } = require("electron");
const MainScreen = require("./screens/main/mainScreen");
const Globals = require("./globals");
const { autoUpdater, AppUpdater } = require("electron-updater");
const log = require('electron-log');
autoUpdater.logger = log;
autoUpdater.logger.transports.file.level = 'info';

let curWindow;

//Basic flags
autoUpdater.autoDownload = true;
autoUpdater.autoInstallOnAppQuit = true;

function createWindow() {
  curWindow = new MainScreen();
}

app.whenReady().then(() => {
  createWindow();

  app.on("activate", function () {
    if (BrowserWindow.getAllWindows().length == 0) createWindow();
  });

  // autoUpdater.checkForUpdates();
  autoUpdater.checkForUpdatesAndNotify()
  curWindow.showMessage(`Checking for updates. Current version ${app.getVersion()}`);
});

/*New Update Available*/
autoUpdater.on("update-available", (info) => {
  curWindow.showMessage(`Update available. Current version ${app.getVersion()}, ${JSON.stringify(info)}`);
  let pth = autoUpdater.downloadUpdate();
  curWindow.showMessage(pth);
});

autoUpdater.on("update-not-available", (info) => {
  curWindow.showMessage(`No update available. Current version ${app.getVersion()}, ${JSON.stringify(info)}`);
});

/*Download Completion Message*/
autoUpdater.on("update-downloaded", (info) => {
  curWindow.showMessage(`Update downloaded. Current version ${app.getVersion()}, ${JSON.stringify(info)}`);
  autoUpdater.quitAndInstall()
});

autoUpdater.on("error", (info) => {
  curWindow.showMessage(info);
});




//Global exception handler
process.on("uncaughtException", function (err) {
  console.log(err);
});

app.on("window-all-closed", function () {
  if (process.platform != "darwin") app.quit();
});
```
