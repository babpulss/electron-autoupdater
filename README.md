# 일렉트론 배포서버를 깃헙이 아닌 로컬호스트로 설정하기

1. 일렉트론 업데이터가 작동하는 흐름을 대강 잡으니까 구현하기가 쉬워졌다. 최신 버전을 검증하는 과정은 서버가 아닌 클라이언트였다.
2. 따라서 서버는 yml, blockmap과 exe에 대한 최신버전만 잘 가지고 제때 제때 클라이언트에게 전달만 해주면 그만이다.
  -   autoUpdater.checkForUpdatesAndNotify()를 실행하면 package.json에서 설정한 빌드.퍼블리시.url로 요청을 하되 디폴트 채널값이 latest.yml의 주소인 localhost:3000/download/latest.yml 요청한다
  -   latest.yml로 버전을 비교한 뒤 버전이 맞지 않으면 같은 경로로 blockmap을 요청하고 체크섬 검증 후 실제 exe파일을 요청한다.
  -   다운로드한 파일은 C:\Users\\${user}\AppData\Local\\${appName}에 저장된다.
  -   express로 로컬서버를 실행한 후 서버로 요청되는 파라미터를 출력하면 같은 경로를 통해 순차적으로 파일을 요청하는 사실을 알 수 있다.

```node
app.get('/download/:fileName', (req, res) => {
  console.log(req.params);
    return res.download(`./files/${req.params.fileName}`)
})
```
```log
{ fileName: 'latest.yml' }
{ fileName: 'Autoupdater app Setup 1.0.0.exe.blockmap' }
Error: ENOENT: no such file or directory, stat 'C:\scln\simple-update-server\files\Autoupdater app Setup 1.0.0.exe.blockmap'
{ fileName: 'Autoupdater app Setup 2.0.0.exe.blockmap' }
{ fileName: 'Autoupdater app Setup 1.0.0.exe.blockmap' }
{ fileName: 'Autoupdater app Setup 2.0.0.exe.blockmap' }
Error: ENOENT: no such file or directory, stat 'C:\scln\simple-update-server\files\Autoupdater app Setup 1.0.0.exe.blockmap'
{ fileName: 'Autoupdater app Setup 2.0.0.exe' }
{ fileName: 'Autoupdater app Setup 2.0.0.exe' }
{ fileName: 'latest.yml' }
```

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
        "url": "http://localhost:3030/download",
        "channel": "latest"
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
