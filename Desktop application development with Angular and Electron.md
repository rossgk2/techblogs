# Desktop application development with Angular and Electron

[Electron](https://www.electronjs.org/docs/latest/) is a framework that allows you to use web development paradigms (i.e. HTML, CSS, JavaScript) to develop cross-platform desktop applications. Typically, desktop applications are developed in lower-level, compiled languages such as Java, C#, and C++, so it's neat that this is possible.

It's simple enough to create an Electron application that uses "vanilla" HTML, CSS, and JavaScript, but what if we want to use a web development framework, like Angular or React? 

In this article, I'll show how this is done. We'll walk through setting up an Electron-backend Angular-frontend desktop application.

## Node and npm

Electron runs on node.js. (If you don't already know about node.js, know that node.js is basically "JavaScript that runs on a server instead of in a browser".)

Since this is the case, we'll need to install Node Package Manager, or npm. If you don't already have npm, an easy way to obtain it is to run one of the Node.js installer executables. Running such an executable will install both Node.js and npm.

## Angular

- Once npm is installed, we can use it to install Angular. Open a command prompt and run `npm install -g @angular/cli` to do so. After this command finishes executing, you can sanity-check that Angular is indeed installed by executing `ng version`; this should display some information on the version of Angular you now have installed.
- Navigate to wherever you want to store the project folder for the demo application we will be creating. Then execute `ng new ae_demo`. Choose `y` when asked if you want to add Angular routing (we won't be making use of this feature, but in general, you probably do want to use it), and choose `CSS` when asked what stylesheet format should be used (for simplicity). 
- This creates a project folder titled ae_demo and fills it with several files. The ae_demo folder and the files it contains together constitute an Angular project, i.e. an Angular webapp.
- To run the Angular webapp, execute `cd ae_demo` and then `ng serve`. After the output from `ng serve` indicates that the webapp is live, visit the webapp yourself by navigating to http://localhost:4200/ in your web browser. You can also stop the webapp by pressing CTRL + C while your cursor is captured by the command line window.
- At this point, we could spend a lot of time learning how to write Angular code and thereby increase the sophistication of our webapp. For the purposes of this tutorial, here's an overview of the relevant parts of writing Angular code:
  - You develop your Angular app by modifying the files in the ae_demo/src/app folder.
  - When your development is complete, you run `ng build`. This produces a subfolder of ae_demo named dist that contains an `index.html` file, a `.css` file, and three `.js` files. So, at the end of the day, your Angular app complies to a regular webapp that uses JavaScript. These are the files that you would provide to a production webserver if you wanted to run your app on it.

## Electron

- Now that we have an idea of how to write the frontend of our application in Angular, we will learn how to use Electron for the backend. The first step is of course to install Electron with npm: `cd` into ae_demo and execute `npm install electron --save-dev`.

- Now, within ae_demo/src, create a folder called electron. In the electron folder, create a file called main.js with the following contents:

  ```typescript
  /* ========================= */
  /* main.js */
  /* ========================= */
  
  const {app, BrowserWindow} = require('electron');
  const url = require("url");
  const path = require("path");
  
  let mainWindow; // of type BrowserWindow
  
  /* Loads the index.html file into the window win. */
  function loadIndexHtml(win) {
    win.loadURL(
      url.format({
        pathname: path.join(__dirname, `../../dist/ae_demo/index.html`),
        protocol: "file:",
        slashes: true
      })
    );
  }
  
  /* Configures the window win so that the renderer process (i.e. the Angular scripts) is loaded only after "DOMContentLoaded" occurs. This prevents us from getting "is not a function" errors when using functions exposed from Electron in Angular scripts. */
  function configLoadRendererAfterDOMContentLoaded(win) {
    /* In the anonymous function, we use a common pattern for manually bootstrapping an Angular app. Google "manually boostrapping Angular app" to learn more about this. */
    win.webContents.on("dom-ready", function() {
      const jsCode = `document.addEventListener('DOMContentLoaded', function() { 
        platformBrowserDynamic().bootstrapModule(AppModule).catch(err => console.error(err)); });`
      win.webContents.executeJavaScript(jsCode);
    });
  }
  
  function createWindow () {
    mainWindow = new BrowserWindow({
      width: 800,
      height: 600,
      webPreferences: {
        nodeIntegration: true //necessary?
      }
    });
  
    mainWindow.on("closed", function () { mainWindow = null });
    configLoadRendererAfterDOMContentLoaded(mainWindow);
    loadIndexHtml(mainWindow);
    mainWindow.webContents.openDevTools();
  }
  
  app.whenReady().then(function() {  
    createWindow();
  })
  
  app.on('window-all-closed', function () {
    if (process.platform !== "darwin") app.quit();
  });
  
  app.on('activate', function () {
    if (mainWindow === null) createWindow();
  });
  ```

- Once this is done, you should edit the node.js configuration file, package.json, so that the JSON within is of the following form:

  ```javascript
  {
    ...,
    "main": "./src/electron/main.js",
    "scripts": {
      ...,
      "devStart": "ng build --base-href . && electron .",
      ...
     },
     ...
  }  
  ```

- You can now build and run your Angular-frontend Electron-backend app by executing the following sequence of commands:

  - `cd ae_demo`
  - `ng build --base-href .` (as described before, this produces a folder named `dist` containing the `.html`, `.css`, and `.js` files that constitute the frontend)
  - `./node_modules/.bin/electron .`  (this causes Electron to search for the `dist` folder in the current folder; once it finds this folder, it will run the app constituted by these files)

- Alternatively, you can build and run the app by executing `npm run devStart`.

  - One oddity is we will get an error if we define `devStart` to be `"ng build --base-href . && electron node_modules/.bin/electron ."`; specifying the explicit path to the Electron executable does not work! Only the implicit path causes no error, for some reason.

# Invoking the backend in frontend code

Sometimes we need to be able to perform tasks in the Angular frontend (e.g. make a HTTP request) that typically only the Electron backend can do.

To achieve this, we will define functions that perform the desired tasks in the Electron backend and specify in a new file, ae_demo/electron/preload.js, that these functions are to be made available to the frontend.

For example, suppose we want to use the axios library in node.js to perform a HTTP request.

To do this, we should add the following to main.js:

```javascript
app.whenReady().then(function() {  
  ipcMain.handle("httpRequest1", httpRequest);
}

async function httpRequest(event, requestConfig) {
  return (await axios(requestConfig)).data;
}
```

Then create a file called preload.js in the electron folder that has the following contents:

```typescript
/* ========================= */
/* preload.js                */
/* ========================= */

const { contextBridge, ipcRenderer } = require("electron")

contextBridge.exposeInMainWorld("api", { 
  httpRequest2: (requestConfig) => ipcRenderer.invoke("httpRequest1", requestConfig)
});
```

Add `preload: path.join(__dirname, "preload.js")` to `webPreferences` in the `createWindow()` function of `main.js`:

```javascript
function createWindow () {
  mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      	nodeIntegration: true,
		preload: path.join(__dirname, "preload.js")
    }
  });
```

Now, to make a HTTP request in our Angular code, we just do the following:

```typescript
(<any> window).api.httpRequest2(requestConfig)
```

This works because:

* The preload script exposes an `api` object that has a function called `httpRequest2` to the Angular frontend.
* In the preload script, it is specified that `httpRequest2(requestConfig)` is equal to the result of sending  the `requestConfig` object as a *message* to the *channel* named `httpRequest1`.
* Calling `ipcMain.handle("httpRequest1", httpRequest)` in `main.js` specifies that whenever the `httpRequest1` channel receives an object `obj` as a message, it should call `httpRequest(obj)`.

Thus, we can call`(<any> window).api.httpRequest2(requestConfig)` in Angular code as a means to effectively call `httpRequest(obj)`.
