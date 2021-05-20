# Web-ext

Mozilla's web-ext command line tool with added support for SmartCookieWeb Preview.

## Documentation

* [Getting started with web-ext][web-ext-user-docs]
* [Command reference](https://extensionworkshop.com/documentation/develop/web-ext-command-reference)

Here are the commands you can run. Click on each one for detailed documentation or use `--help` on the command line, such as `web-ext build --help`.

* [`run`](https://extensionworkshop.com/documentation/develop/web-ext-command-reference#web-ext-run)
  * Run the extension
* [`lint`](https://extensionworkshop.com/documentation/develop/web-ext-command-reference#web-ext-lint)
  * Validate the extension source
* [`sign`](https://extensionworkshop.com/documentation/develop/web-ext-command-reference#web-ext-sign)
  * Sign the extension so it can be installed in Firefox
* [`build`](https://extensionworkshop.com/documentation/develop/web-ext-command-reference#web-ext-build)
  * Create an extension package from source
* [`docs`](https://extensionworkshop.com/documentation/develop/web-ext-command-reference#web-ext-docs)
  * Open the `web-ext` documentation in a browser

## Installation from source

You'll need:
* [Node.js](https://nodejs.org/en/), 12.0.0 or higher
* [npm](https://www.npmjs.com/), 6.9.0 or higher is recommended

Optionally, you may like:
* [nvm](https://github.com/creationix/nvm), which helps manage node versions

If you had already installed `web-ext` from npm,
you may need to uninstall it first:

    npm uninstall --global web-ext

Change into the source and install all dependencies:

    git clone https://github.com/CookieJarApps/web-ext.git
    cd web-ext
    npm install

Build the command:

    npm run build

Link it to your node installation:

    npm link

You can now run it from any directory:

    web-ext --help

To get updates, just pull changes and rebuild the executable. You don't
need to relink it.

    cd /path/to/web-ext
    git pull
    npm run build

## Using web-ext in NodeJS code

Aside from [using web-ext on the command line][web-ext-user-docs], you may wish to execute `web-ext` in NodeJS code. There is limited support for this. Here are some examples.

You are able to execute command functions without any argument validation. If you want to execute `web-ext run` you would do so like this:

```js
// const webExt = require('web-ext');
// or...
import webExt from 'web-ext';

webExt.cmd.run({
  // These are command options derived from their CLI conterpart.
  // In this example, --source-dir is specified as sourceDir.
  firefox: '/path/to/Firefox-executable',
  sourceDir: '/path/to/your/extension/source/',
}, {
  // These are non CLI related options for each function.
  // You need to specify this one so that your NodeJS application
  // can continue running after web-ext is finished.
  shouldExitProgram: false,
})
  .then((extensionRunner) => {
    // The command has finished. Each command resolves its
    // promise with a different value.
    console.log(extensionRunner);
    // You can do a few things like:
    // extensionRunner.reloadAllExtensions();
    // extensionRunner.exit();
  });
```

If you would like to run an extension on Firefox for Android:

```js
// Path to adb binary (optional parameter, auto-detected if missing)
const adbBin = "/path/to/adb";
// Get an array of device ids (Array<string>)
const deviceIds = await webExt.util.adb.listADBDevices(adbBin);
const adbDevice = ...
// Get an array of Firefox APKs (Array<string>)
const firefoxAPKs = await webExt.util.adb.listADBFirefoxAPKs(
  deviceId, adbBin
);
const firefoxApk = ...

webExt.cmd.run({
  target: 'firefox-android',
  firefoxApk,
  adbDevice,
  sourceDir: ...
}).then((extensionRunner) => {...});
```

If you would like to control logging, you can access the logger object. Here is an example of turning on verbose logging:

```js
webExt.util.logger.consoleStream.makeVerbose();
webExt.cmd.run({sourceDir: './src'}, {shouldExitProgram: false});
```

You can also disable the use of standard input:

```js
webExt.cmd.run({noInput: true}, {shouldExitProgram: false});
```

`web-ext` is designed for WebExtensions but you can try disabling manifest validation to work with legacy extensions. This is not officially supported.

```js
webExt.cmd.run(
  {sourceDir: './src'},
  {
    getValidatedManifest: () => ({
      name: 'some-fake-name',
      version: '1.0.0',
    }),
    shouldExitProgram: false,
  },
);
```
