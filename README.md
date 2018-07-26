# ūsus

[![Travis build status](http://img.shields.io/travis/gajus/usus/master.svg?style=flat-square)](https://travis-ci.org/gajus/usus)
[![Coveralls](https://img.shields.io/coveralls/gajus/usus.svg?style=flat-square)](https://coveralls.io/github/gajus/usus)
[![NPM version](http://img.shields.io/npm/v/usus.svg?style=flat-square)](https://www.npmjs.org/package/usus)
[![Canonical Code Style](https://img.shields.io/badge/code%20style-canonical-blue.svg?style=flat-square)](https://github.com/gajus/canonical)
[![Twitter Follow](https://img.shields.io/twitter/follow/kuizinas.svg?style=social&label=Follow)](https://twitter.com/kuizinas)

Webpage pre-rendering service. ⚡️

## Deprecated

Deprecated in favour of https://github.com/GoogleChrome/rendertron

The two projects are not functionally interchangeable. However, the overlap is significant enough to discourage the duplication of effort.

Future releases of ūsus will be limited to bug fixes.

## Features

* Renders webpage using the [Chrome Debugging Protocol](https://chromedevtools.github.io/devtools-protocol/) (CDP).
* Extracts CSS used to render the page.
* Renders HTML with the blocking CSS made asynchronous.
* Inlines the critical CSS.
* Preloads CSS and fonts used to render the page using `rel=preload`.

---

* [Motivation](#motivation)
  * [Demo](#demo)
  * [Use cases](#use-cases)
* [Node.js API](#nodejs-api)
  * [Configuration](#configuration)
* [Installation](#installation)
  * [Dependencies](#dependencies)
* [Cookbook](#cookbook)
  * [Using via the command line interface (CLI)](#using-via-the-command-line-interface-cli)
  * [Building Docker container with Chrome](#building-docker-container-with-chrome)
  * [Minify the CSS](#minify-the-css)
  * [Debugging](#debugging)
* [Implementation](#implementation)
* [Alternatives](#alternatives)

## Motivation

Static HTML pages with inline CSS load faster and are better indexed than single page applications (SPA). ūsus pre-renders single page applications into static HTML with the critical CSS inlined.

Removing the blocking CSS and inlining the CSS required to render the page increases the perceived page loading speed. Presumably, improves SEO by reducing the page loading time.

Read [Pre-rendering SPA for SEO and improved perceived page loading speed](https://medium.com/@gajus/pre-rendering-spa-for-seo-and-improved-perceived-page-loading-speed-47075aa16d24).

### Demo

Examples of web pages using ūsus:

* https://go2cinema.com/movies/baywatch-2017-1198354
* https://go2cinema.com/venues/odeon-leicester-square-1001206

### Use cases

* Produce HTML used to render the page. Used to render single page applications (e.g. React and Angular) to produce a static HTML. This can be used as a replacement of https://prerender.io/. Default behaviour.
* Extract CSS used to render a specific page. Used to capture the critical CSS. Use `--extractStyles` option.
* Produce HTML used to render the page with the critical-path CSS inlined and blocking CSS made asynchronous. Use `--inlineStyles` option.

## Node.js API

ūsus can be used either as a Node.js dependency or as a [CLI program](#using-via-the-command-line-interface-cli).

```js
import {
  render
} from 'usus';

/**
 * @see https://github.com/gajus/usus#configuration
 */
const configuration: UserConfigurationType = {}

const css = await render('http://gajus.com/', configuration);

```

### Configuration

```js
// Flow type annotations included for user reference only.
// ūsus does not depend or require use of Flow type.
//
// Refer to the table below for an alternative form of documentation.

type CookieType = {|
  +name: string,
  +value: string
|};

export type UserDeviceMetricsOverrideType = {
  +deviceScaleFactor?: number,
  +fitWindow?: boolean,
  +height?: number,
  +mobile?: boolean,
  +width?: number
};

type FormatStylesType = (styles: string) => Promise<string>;

export type UserConfigurationType = {
  +chromePort?: number,
  +cookies?: $ReadOnlyArray<CookieType>,
  +delay?: number,
  +deviceMetricsOverride?: UserDeviceMetricsOverrideType,
  +extractStyles?: boolean,
  +formatStyles?: FormatStylesType,
  +inlineStyles?: boolean,
  +preloadFonts?: boolean,
  +preloadStyles?: boolean
};

```

The default behaviour is to return the HTML.

* Using the `--extractStyles` option returns the CSS used to render the document.
* Using the `--inlineStyles` option returns HTML document with CSS inlined.

|Name|Type|Description|Default value|
|---|---|---|---|
|`chromePort`|`number`|Port of an existing Chrome instance. See [Controlling the Chrome instance](#controlling-the-chrome-instance).|N/A|
|`cookies`|`Array<{name: string, value: string}>`|Sets a cookie with the given cookie data.|N/A|
|`delay`|`number`|Defines how many milliseconds to wait after the "load" event has been fired before capturing the styles used to load the page. This is important if resources appearing on the page are being loaded asynchronously.|`number`|`5000`|
|`deviceMetricsOverride`||See [`deviceMetricsOverride` configuration](#devicemetricsoverride-configuration)||
|`extractStyles`|`boolean`|Extracts CSS used to render the page.|`false`|
|`formatStyles`|`(styles: string) => Promise<string>`|Used to format CSS. Useful with `inlineStyles=true` option to format the CSS before it is inlined.|N/A|
|`inlineStyles`|`boolean`|Inlines the styles required to render the document.|`false`|
|`preloadFonts`|`boolean`|Adds `rel=preload` for all fonts required to render the page.|`true`|
|`preloadStyles`|`boolean`|Adds `rel=preload` for all styles removed from `<head>`. Used with `inlineStyles=true`.|`true`|
|`url`|`string`|The URL to render.|N/A|

#### `deviceMetricsOverride` configuration

|Name|Type|Description|Default value|
|---|---|---|---|
|`deviceScaleFactor`|`number`|Overriding device scale factor value.|`1`|
|`fitWindow`|`boolean`|Whether a view that exceeds the available browser window area should be scaled down to fit.|`false`|
|`height`|`number`|Overriding width value in pixels (minimum 0, maximum 10000000).|`1080`|
|`width`|`number`|Overriding height value in pixels (minimum 0, maximum 10000000).|`1920`|
|`mobile`|`boolean`|Whether to emulate mobile device. This includes viewport meta tag, overlay scrollbars, text autosizing and more.|`false`|

For more information about the `deviceMetricsOverride` configuration, refer to [Chrome DevTools Protocol Viewer documentation](https://chromedevtools.github.io/devtools-protocol/tot/Emulation/#method-setDeviceMetricsOverride).

## Installation

Using ūsus requires to install [`usus` NPM](https://www.npmjs.com/package/usus) package and [Google Chrome](https://www.google.com/chrome/index.html) browser (refer to [Dependencies](#dependencies)).

### Dependencies

ūsus depends on [Chrome v59+](https://developers.google.com/web/updates/2017/04/headless-chrome).

For Docker installation instructions, refer to [Building Docker container with Chrome](#building-docker-container-with-chrome).

## Cookbook

### Using via the command line interface (CLI)

```bash
$ npm install usus --global
$ usus --help
# usus <command> --help
$ usus render --help
# Renders static HTML. Equivalent to https://prerender.io/.
$ usus render --url http://gajus.com/
# Inlines styles required to render the page.
$ usus render --url http://gajus.com/ --inlineStyles true
# Use cookies when loading the page.
$ usus render --url http://gajus.com/ --cookies foo=bar,baz=qux
# Render emulating a mobile device (example is using iPhone 6 parameters).
$ usus render --url http://gajus.com/ --deviceMetricsOverride.deviceScaleFactor 2 --deviceMetricsOverride.fitWindow false --deviceMetricsOverride.height 1334 --deviceMetricsOverride.mobile true --deviceMetricsOverride.width 750

```

### Building Docker container with Chrome

Add the following line to your `Dockerfile`:

```bash
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
  && sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' \
  && apt-get update -y \
  && apt-get install google-chrome-stable -y

```

This assumes that you are extending from the base [`node` image](https://github.com/nodejs/docker-node).

### Controlling the Chrome instance

By default, ūsus creates a new instance of Chrome for every `render` operation and destroys it after completion. However, you can start Chrome independent of ūsus and re-use the same instance for multiple renderings.

```js
import {
  launchChrome,
  render
} from 'usus';

const chrome = await launchChrome();

await render('https://go2cinema.com/movies/baywatch-2017-1198354', {
  chromePort: chrome.port,
  inlineStyles: true
});

await render('https://go2cinema.com/movies/baby-driver-2017-2257838', {
  chromePort: chrome.port,
  inlineStyles: true
});

await chrome.kill();

```

`launchChrome` is a convenience method to launch Chrome using default ūsus configuration. If you need granular control over how Chrome is launched, refer to the [chrome-launcher](https://github.com/GoogleChrome/lighthouse/tree/master/chrome-launcher) program.

### Minifying the CSS

Use the `formatStyles` callback to minify/ format/ optimize/ remove CSS before it is inlined.

In this example, I am using [`csso`](https://github.com/css/csso) minifier.

```js
import {
  render
} from 'usus';
import {
  minify
} from 'csso';

await render(url, {
  formatStyles: (styles: string): Promise<string> => {
    return minify(styles).css;
  },
  inlineStyles: true
});

```

### Debugging

Export `DEBUG=usus` variable to get additional debugging information, e.g.

```bash
$ export DEBUG=usus*
$ usus --url http://gajus.com

```

## Implementation

ūsus uses [Chrome Debugging Protocol](https://chromedevtools.github.io/devtools-protocol/) [CSS coverage report](https://chromedevtools.github.io/devtools-protocol/tot/CSS/#method-takeCoverageDelta) to generate a stylesheet used to render the document.

## Alternatives

The following programs provide equivalent service:

* https://github.com/giakki/uncss
* https://github.com/pocketjoso/penthouse
* https://github.com/addyosmani/critical
* https://github.com/filamentgroup/criticalcss

All of these programs are using PhantomJS or JSDom to render the page/ evaluate the scripts.

ūsus is different because it is using [Chrome Debugging Protocol](https://chromedevtools.github.io/devtools-protocol/) and it leverages the Chrome [CSS coverage report](https://chromedevtools.github.io/devtools-protocol/tot/CSS/#method-takeCoverageDelta) to generate a stylesheet used to render the document.
