# scw-web-ext

Mozilla's web-ext command line tool with added support for SmartCookieWeb Preview and unnecessary code for desktop Firefox removed. Not endorsed by Mozilla in any way. Can be installed alongside web-ext.

## Documentation

### Test add-on
ADB must be installed on your PC, USB Debugging must be enabled on your Android device, and if you're on SmartCookieWeb-Preview v7 or newer, remote debugging must be on in advanced settings.

Run `scw-web-ext run --target=firefox-android` and find your Device ID.

Then paste the ID into this command: `scw-web-ext run --target=firefox-android --android-device=<Device ID>`.

Navigate to the directory your add-on's manifest.json is in and then run `scw-web-ext run --target=firefox-android --android-device=<Device ID> --firefox-apk=com.cookiejarapps.android.smartcookieweb`.

## Installation from npm

First, make sure you are running the current
[LTS](https://github.com/nodejs/LTS)
(long term support) version of
[NodeJS](https://nodejs.org/en/).

### Global command

You can install this command onto your machine globally with:

    npm install --global scw-web-ext

### For your project

Alternatively, you can install this command as one of the
[`devDependencies`](https://docs.npmjs.com/files/package.json#devdependencies)
of your project.  This method can help you control the version of `scw-web-ext`
as used by your team.

    npm install --save-dev web-ext

Next you can use the `web-ext` command in your project as an
[npm script](https://docs.npmjs.com/misc/scripts).
Here is an example where the `--source-dir` argument specifies where to find
the source code for your extension.

`package.json`
```json
"scripts": {
  "start:firefox": "scw-web-ext run --source-dir ./extension-dist/",
}
```

You can always pass in additional commands to your npm scripts using
the `--` suffix. For example, the previous script could specify the Firefox
version on the command line with this:

    npm run start:firefox -- --firefox=nightly

## Installation from source

You'll need:
* [Node.js](https://nodejs.org/en/), 12.0.0 or higher
* [npm](https://www.npmjs.com/), 6.9.0 or higher is recommended

Optionally, you may like:
* [nvm](https://github.com/creationix/nvm), which helps manage node versions

If you had already installed `scw-web-ext` from npm,
you may need to uninstall it first:

    npm uninstall --global scw-web-ext

Change into the source and install all dependencies:

    git clone https://github.com/cookiejarapps/scw-web-ext.git
    cd scw-web-ext
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

Aside from [using web-ext on the command line][web-ext-user-docs], you may wish to execute `scw-web-ext` in NodeJS code. There is limited support for this. Here are some examples.

You are able to execute command functions without any argument validation. If you want to execute `scw-web-ext run` you would do so like this:

```js
// const webExt = require('scw-web-ext');
// or...
import webExt from 'scw-web-ext';

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
