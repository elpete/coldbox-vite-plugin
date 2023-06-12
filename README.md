# ColdBox Vite Plugin

<a href="https://github.com/elpete/coldbox-vite-plugin/actions"><img src="https://github.com/elpete/coldbox-vite-plugin/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://www.npmjs.com/package/coldbox-vite-plugin"><img src="https://img.shields.io/npm/dt/coldbox-vite-plugin" alt="Total Downloads"></a>
<a href="https://www.npmjs.com/package/coldbox-vite-plugin"><img src="https://img.shields.io/npm/v/coldbox-vite-plugin" alt="Latest Stable Version"></a>
<a href="https://www.npmjs.com/package/coldbox-vite-plugin"><img src="https://img.shields.io/npm/l/coldbox-vite-plugin" alt="License"></a>

- [Introduction](#introduction)
- [Installation & Setup](#installation)
  - [Installing Node](#installing-node)
  - [Installing Vite And The ColdBox Plugin](#installing-vite-and-coldbox-plugin)
  - [Configuring Vite](#configuring-vite)
  - [Loading Your Scripts And Styles](#loading-your-scripts-and-styles)
- [Running Vite](#running-vite)
- [Working With JavaScript](#working-with-scripts)
  - [Aliases](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [URL Processing](#url-processing)
- [Working With Stylesheets](#working-with-stylesheets)
- [Working With Blade & Routes](#working-with-blade-and-routes)
  - [Processing Static Assets With Vite](#blade-processing-static-assets)
  - [Refreshing On Save](#refreshing-on-save)
  - [Aliases](#blade-aliases)
- [Custom Base URLs](#custom-base-urls)
- [Environment Variables](#environment-variables)
- [Disabling Vite In Tests](#disabling-vite-in-tests)
- [Server-Side Rendering (SSR)](#ssr)
- [Script & Style Tag Attributes](#script-and-style-attributes)
  - [Content Security Policy (CSP) Nonce](#content-security-policy-csp-nonce)
  - [Subresource Integrity (SRI)](#subresource-integrity-sri)
  - [Arbitrary Attributes](#arbitrary-attributes)
- [Advanced Customization](#advanced-customization)
  - [Correcting Dev Server URLs](#correcting-dev-server-urls)

<a name="introduction"></a>
## Introduction

[Vite](https://vitejs.dev) is a modern frontend build tool that provides an extremely fast development environment and bundles your code for production. When building applications with ColdBox, you will typically use Vite to bundle your application's CSS and JavaScript files into production ready assets.

ColdBox integrates seamlessly with Vite by providing an official plugin and Blade directive to load your assets for development and production.

<a name="vite-or-mix"></a>
#### Choosing Between Vite And ColdBox Elixir

Before transitioning to Vite, new ColdBox applications utilized [ColdBox Elixir](https://coldbox-elixir.ortusbooks.com/), which is powered by [webpack](https://webpack.js.org/), when bundling assets. Vite focuses on providing a faster and more productive experience when building rich JavaScript applications. If you are developing a Single Page Application (SPA), including those developed with tools like [Inertia](https://inertiajs.com), Vite will be the perfect fit.

Vite also works well with traditional server-side rendered applications with JavaScript "sprinkles", including those using [CBWire](https://cbwire.ortusbooks.com). However, it lacks some features that ColdBox Elixir supports, such as the ability to copy arbitrary assets into the build that are not referenced directly in your JavaScript application.

<a name="installation"></a>
## Installation & Setup

> **Note**
> The following documentation discusses how to manually install and configure the ColdBox Vite plugin. However, there is a [ColdBox Vite template](https://github.com/coldbox-templates/vite) and a [Quick Tailwind Inertia template](https://github.com/elpete/quick-tailwind-inertia) that already include all of this scaffolding and are the fastest way to get started with ColdBox and Vite.

<a name="installing-node"></a>
### Installing Node

You must ensure that Node.js (16+) and NPM are installed before running Vite and the ColdBox plugin:

```sh
node -v
npm -v
```

You can easily install the latest version of Node and NPM using simple graphical installers from [the official Node website](https://nodejs.org/en/download/).

<a name="installing-vite-and-coldbox-plugin"></a>
### Installing Vite And The ColdBox Plugin

Within a fresh installation of ColdBox, you may need to create a `package.json` file in the root of your application's directory structure. You can do this using the `npm init` command.

```sh
npm init -y
```

Once you have a `package.json`, install `vite` and `coldbox-vite-plugin` to get started.

```sh
npm install -D vite coldbox-vite-plugin
```

<a name="configuring-vite"></a>
### Configuring Vite

Vite is configured via a `vite.config.js` file in the root of your project. You are free to customize this file based on your needs, and you may also install any other plugins your application requires, such as `@vitejs/plugin-vue` or `@vitejs/plugin-react`.

The ColdBox Vite plugin requires you to specify the entry points for your application. These may be JavaScript or CSS files, and include preprocessed languages such as TypeScript, JSX, TSX, and Sass.

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox([
            "resources/assets/css/app.css",
            "resources/assets/js/app.js",
        ])
    ]
});
```

If you are building an SPA, including applications built using Inertia, Vite works best without CSS entry points:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox([
            "resources/js/app.js",
        ])
    ]
});
```

Instead, you should import your CSS via JavaScript. Typically, this would be done in your application's `resources/assets/js/app.js` file:

```js
import "../css/app.css";
```

The ColdBox plugin also supports multiple entry points and advanced configuration options such as [SSR entry points](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Working With A Secure Development Server

If your local development web server is serving your application via HTTPS, you may run into issues connecting to the Vite development server. You should generate a trusted certificate and manually configure Vite to use the generated certificates:

```js
// ...
import fs from "fs";

const host = "my-app.test";

export default defineConfig({
    // ...
    server: {
        host,
        hmr: { host },
        https: {
            key: fs.readFileSync(`/path/to/${host}.key`),
            cert: fs.readFileSync(`/path/to/${host}.crt`),
        }
    }
});
```

If you are unable to generate a trusted certificate for your system, you may install and configure the [`@vitejs/plugin-basic-ssl` plugin](https://github.com/vitejs/vite-plugin-basic-ssl). When using untrusted certificates, you will need to accept the certificate warning for Vite's development server in your browser by following the "Local" link in your console when running the `npm run dev` command.

<a name="loading-your-scripts-and-styles"></a>
### Loading Your Scripts And Styles

With your Vite entry points configured, you may now reference them in your layout file.  The ColdBox templates mentioned above include a `vite` helper function to generate the correct asset paths:

```cfm
<!doctype html>
<head>
    #vite( "resources/assets/css/app.css" )#
</head>

<body>
    #vite( "resources/assets/js/app.js" )#
</body>
```

If you're importing your CSS via JavaScript, you only need to include the JavaScript entry point:

```cfm
<!doctype html>
<body>
    #vite( "resources/assets/js/app.js" )#
</body>
```

The `vite` function will automatically detect the Vite development server and inject the Vite client to enable Hot Module Replacement. In build mode, the directive will load your compiled and versioned assets, including any imported CSS.

<a name="running-vite"></a>
## Running Vite

There are two ways you can run Vite. You may run the development server, which is useful while developing locally. The development server will automatically detect changes to your files and instantly reflect them in any open browser windows.  The ColdBox templates alias this as a `dev` script in the `package.json`:

```json
{
    "scripts": {
        "dev": "vite"
    }
}
```

Or, running `vite build` will version and bundle your application's assets and get them ready for you to deploy to production. The ColdBox templates alias this as a `build` script in the `package.json`:

```json
{
    "scripts": {
        "build": "vite build"
    }
}
```

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

<a name="working-with-scripts"></a>
## Working With JavaScript

<a name="aliases"></a>
### Aliases

By default, The ColdBox plugin provides a common alias to help you hit the ground running and conveniently import your application's assets:

```js
{
    "@" => "/resources/assets/js"
}
```

You may overwrite the `"@"` alias by adding your own to the `vite.config.js` configuration file:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox(["resources/assets/ts/app.tsx"]),
    ],
    resolve: {
        alias: {
            "@": "/resources/assets/ts",
        }
    }
});
```

<a name="vue"></a>
### Vue

If you would like to build your frontend using the [Vue](https://vuejs.org/) framework, then you will also need to install the `@vitejs/plugin-vue` plugin:

```sh
npm install -D @vitejs/plugin-vue
```

You may then include the plugin in your `vite.config.js` configuration file.

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
    plugins: [
        coldbox(["resources/js/app.js"]),
        vue()
    ]
});
```

<a name="react"></a>
### React

If you would like to build your frontend using the [React](https://reactjs.org/) framework, then you will also need to install the `@vitejs/plugin-react` plugin:

```sh
npm install -D @vitejs/plugin-react
```

You may then include the plugin in your `vite.config.js` configuration file:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";
import react from "@vitejs/plugin-react";

export default defineConfig({
    plugins: [
        coldbox(["resources/js/app.jsx"]),
        react()
    ]
});
```

You will need to ensure that any files containing JSX have a `.jsx` or `.tsx` extension, remembering to update your entry point, if required, as [shown above](#configuring-vite).

<a name="inertia"></a>
### Inertia

The ColdBox Vite plugin provides a convenient `resolvePageComponent` function to help you resolve your Inertia page components. Below is an example of the helper in use with Vue 3; however, you may also utilize the function in other frameworks such as React:

```js
import { createApp, h } from "vue";
import { createInertiaApp } from "@inertiajs/vue3";
import { resolvePageComponent } from "coldbox-vite-plugin/inertia-helpers";

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob("./Pages/**/*.vue")),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  }
});
```

<a name="url-processing"></a>
### URL Processing

When using Vite and referencing assets in your application's HTML, CSS, or JS, there are a couple of caveats to consider. First, if you reference assets with an absolute path, Vite will not include the asset in the build; therefore, you should ensure that the asset is available in your public directory.

When referencing relative asset paths, you should remember that the paths are relative to the file where they are referenced. Any assets referenced via a relative path will be re-written, versioned, and bundled by Vite.

Consider the following project structure:

```nothing
includes/
  coldbox.png
resources/
  assets/
    js/
      Pages/
        App.vue
  images/
    testbox.png
```

The following example demonstrates how Vite will treat relative and absolute URLs:

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/includes/coldbox.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/testbox.png">
```

<a name="working-with-stylesheets"></a>
## Working With Stylesheets

You can learn more about Vite's CSS support within the [Vite documentation](https://vitejs.dev/guide/features.html#css). If you are using PostCSS plugins such as [Tailwind](https://tailwindcss.com), you may create a `postcss.config.js` file in the root of your project and Vite will automatically apply it:

```js
module.exports = {
    plugins: {
        tailwindcss: {},
        autoprefixer: {}
    }
};
```

<a name="refreshing-on-save"></a>
### Refreshing On Save

When your application is built using traditional server-side rendering, Vite can improve your development workflow by automatically refreshing the browser when you make changes to view files in your application. To get started, you can simply specify the `refresh` option as `true`.

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox({
            // ...
            refresh: true,
        })
    ]
});
```

When the `refresh` option is `true`, saving files in the following directories will trigger the browser to perform a full page refresh while you are running `npm run dev`:

- `handlers/**`
- `models/**`
- `layouts/**`
- `views/**`
- `config/**`

If these default paths do not suit your needs, you can specify your own list of paths to watch:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox({
            // ...
            refresh: ["some/custom/path/**""],
        })
    ]
});
```

Under the hood, the ColdBox Vite plugin uses the [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload) package, which offers some advanced configuration options to fine-tune this feature's behavior. If you need this level of customization, you may provide a `config` definition:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox({
            // ...
            refresh: [{
                paths: ["path/to/watch/**"],
                config: { delay: 300 }
            }]
        })
    ]
});
```

<a name="custom-base-urls"></a>
## Custom Base URLs

If your Vite compiled assets are deployed to a domain separate from your application, such as via a CDN, you must specify the `ASSET_URL` environment variable within your application's `.env` file:

```env
ASSET_URL=https://cdn.example.com
```

After configuring the asset URL, all re-written URLs to your assets will be prefixed with the configured value:

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Remember that [absolute URLs are not re-written by Vite](#url-processing), so they will not be prefixed.

<a name="environment-variables"></a>
## Environment Variables

You may inject environment variables into your JavaScript by prefixing them with `VITE_` in your application's `.env` file:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

You may access injected environment variables via the `import.meta.env` object:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="ssr"></a>
## Server-Side Rendering (SSR)

The ColdBox Vite plugin makes it painless to set up server-side rendering with Vite. To get started, create an SSR entry point at `resources/assets/js/ssr.js` and specify the entry point by passing a configuration option to the ColdBox plugin:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox({
            input: "resources/assets/js/app.js",
            ssr: "resources/assets/js/ssr.js",
        })
    ]
});
```

To ensure you don't forget to rebuild the SSR entry point, we recommend augmenting the "build" script in your application's `package.json` to create your SSR build:

```diff
  "scripts": {
      "dev": "vite",
-     "build": "vite build"
+     "build": "vite build && vite build --ssr"
  }
```

Then, to build and start the SSR server, you may run the following commands:

```sh
npm run build
node bootstrap/ssr/ssr.mjs
```

<a name="advanced-customization"></a>
## Advanced Customization

Out of the box, ColdBox's Vite plugin uses sensible conventions that should work for the majority of applications; however, sometimes you may need to customize Vite's behavior. Within the `vite.config.js` file, you should then specify many overrides:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";

export default defineConfig({
    plugins: [
        coldbox({
            hotFile: "storage/vite.hot", // Customize the "hot" file...
            buildDirectory: "bundle", // Customize the build directory...
            input: ["resources/js/app.js"], // Specify the entry points...
        })
    ]
    build: {
      manifest: "assets.json", // Customize the manifest filename...
    }
});
```

<a name="correcting-dev-server-urls"></a>
### Correcting Dev Server URLs

Some plugins within the Vite ecosystem assume that URLs which begin with a forward-slash will always point to the Vite dev server. However, due to the nature of the ColdBox integration, this is not the case.

For example, the `vite-imagetools` plugin outputs URLs like the following while Vite is serving your assets:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

The `vite-imagetools` plugin is expecting that the output URL will be intercepted by Vite and the plugin may then handle all URLs that start with `/@imagetools`. If you are using plugins that are expecting this behaviour, you will need to manually correct the URLs. You can do this in your `vite.config.js` file by using the `transformOnServe` option.

In this particular example, we will prepend the dev server URL to all occurrences of `/@imagetools` within the generated code:

```js
import { defineConfig } from "vite";
import coldbox from "coldbox-vite-plugin";
import { imagetools } from "vite-imagetools";

export default defineConfig({
    plugins: [
        coldbox({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll("/@imagetools", devServerUrl+"/@imagetools"),
        }),
        imagetools()
    ]
});
```

Now, while Vite is serving Assets, it will output URLs that point to the Vite dev server:

```diff
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

## License

The ColdBox Vite plugin is open-sourced software licensed under the [MIT license](LICENSE.md).
