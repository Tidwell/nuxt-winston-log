# nuxt-winston-log

<img alt="Winston + Nuxt logo" src="https://raw.githubusercontent.com/aaronransley/nuxt-winston-log/master/icon.png" width="128" />

A module to add `winston` / logging to your Nuxt application. This module only supports Nuxt apps running in universal mode.

By default the following events are captured:

1. `error` level: SSR errors via Nuxt middleware hooks
3. `info` level: Basic access logs for `serverMiddleware` endpoints + pages in your Nuxt app

All logs captured include a bit of additional metadata pulled from the [Node.js `request` object](https://nodejs.org/dist/latest-v10.x/docs/api/http.html#http_class_http_incomingmessage):

```js
{
  url: 'https://cool.net',
  method: 'GET',
  headers: {
    'X-Plumbus': "36f7b241-2910-4439-8671-749fc77dc213"
  }
}
```

Logs are output at `./logs/{NODE_ENV}.log` by default. They are created in [JSON Lines](http://jsonlines.org/) format.

# Installation

1. Install npm package

```sh
$ yarn add nuxt-winston-log # or npm i nuxt-winston-log
```

2. Edit your `nuxt.config.js` file to add module

```js
{
  modules: ['nuxt-winston-log']
}
```

3. Change options as needed. See Usage section for details.

# Usage

1. By default, `nuxt-winston-log` exposes some basic options for common needs. Internally, these options are used to create a basic [Logger instance](https://github.com/winstonjs/winston#creating-your-own-logger) and wire up middleware.

    The default values are:

```js
{
  // Path that log files will be created in.
  // Change this to keep things neat.
  logPath: './logs',
  // Name of log file.
  // Change this to keep things tidy.
  logName: `${process.env.NODE_ENV}.log`
}
```

If you pass `useFileLogging: false` in the config options, it will ignore all of
the file creation and not force winston to create the log folder.

2. To customize the [File Transport instance](https://github.com/winstonjs/winston/blob/master/docs/transports.md#file-transport), pass options to the `transportOptions` key:

```js
import path from 'path'
const logfilePath = path.resolve(process.cwd(), './logs', `${process.env.NODE_ENV}.log`)

export default {
  winstonLog: {
    transportOptions: {
      filename: logfilePath
    }
  }
}
```

3. To customize the [Logger instance](https://github.com/winstonjs/winston#creating-your-own-logger), pass options to the `loggerOptions` key. Note that you can completely overwrite all defaults this way, and establish your own `format`, `transports`, and so on.

```js
import { format, transports } from 'winston'
const { combine, timestamp, label, prettyPrint } = format

export default {
  winstonLog: {
    loggerOptions: {
      format: combine(
        label({ label: 'Custom Nuxt logging!' }),
        timestamp(),
        prettyPrint()
      ),
      transports: [new transports.Console()]
    }
  }
}
```

4. To access the winston logger instance from within Nuxt lifecycle areas, use the `$winstonLog` key from the Nuxt `context` object.

```js
// e.g. inside `~/store/index.js`
// ...
export const actions = {
  async nuxtServerInit({ store, commit }, { req, $winstonLog }) {
    // ...
    $winstonLog.info(`x-forwarded-host: ${req.headers['x-forwarded-host']}`)
    // ...
  }
}
// ...
```

5. By default the plugin checks headers for requests that pass a `text/html` or json header and only logs those requests.  Set `ignoreHeaderChecks` in nuxt.config.js to log all requests and ignore checking for the headers.

# [Changelog](./CHANGELOG.md)
