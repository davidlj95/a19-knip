# A19Knip

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 19.0.2.

Purpose of is to improve [`knip`](https://knip.dev/) defaults for Angular projects. So that it works nicely out-of-the-box for Angular projects created with default options and server side rendering (SSR) enabled.

## Knip

### Setup

`knip` was added following the [getting started guide](https://knip.dev/overview/getting-started).

```shell
pnpm create @knip/config
```

> [`@knip/create-config`](https://www.npmjs.com/package/@knip/create-config) version `1.0.3`
>
> [`knip`](https://www.npmjs.com/package/knip) version `5.38.3`

### Initial results

After installing `knip`, following output was given:

#### Regular mode

> ##### Unused files (4)
>
> * src/app/app.component.spec.ts
> * src/app/app.config.server.ts
> * src/app/app.routes.server.ts
> * src/main.server.ts
>
> ##### Unused dependencies (3)
>
> | Name                              | Location     | Severity |
> | :-------------------------------- | :----------- | :------- |
> | @angular/platform-browser-dynamic | package.json | error    |
> | @angular/compiler                 | package.json | error    |
> | @angular/forms                    | package.json | error    |
>
> ##### Unused devDependencies (5)
>
> | Name                        | Location     | Severity |
> | :-------------------------- | :----------- | :------- |
> | karma-jasmine-html-reporter | package.json | error    |
> | karma-chrome-launcher       | package.json | error    |
> | karma-coverage              | package.json | error    |
> | karma-jasmine               | package.json | error    |
> | jasmine-core                | package.json | error    |

#### Production mode

> ##### Unused files (4)
>
> * src/app/app.component.spec.ts
> * src/app/app.config.server.ts
> * src/app/app.routes.server.ts
> * src/main.server.ts
>
> ##### Unused dependencies (3)
>
> | Name                              | Location     | Severity |
> | :-------------------------------- | :----------- | :------- |
> | @angular/platform-browser-dynamic | package.json | error    |
> | @angular/compiler                 | package.json | error    |
> | @angular/forms                    | package.json | error    |
>
> ##### Unlisted binaries (1)
>
> | Name | Location     | Severity |
> | :- | :----------- | :------- |
> | ng | package.json | error    |

### Improvements

There are several improvements that can be grouped by category:

#### Server Side Rendering (SSR)

##### Include `src/main.server.ts`

As production entry file (comes from `angular.json` `projects.*.architect.build.options.server` when using the new ESBuild-based `application` builder).

This will then use `src/app/app.config.server.ts`. Which will then use the `serverRoutes` from `src/app/app.routes.server.ts`. So those unused reported files will be gone too.

#### Testing with Karma & Jasmine

By default, Angular CLI initializes a workspace with an application with unit testing provided by Karma & Jasmine. This is specified in the Angular workspace configuration file `angular.json`, under the test target for a project: `projects.*.architect.test`. The builder is [`@angular-devkit/build-angular:karma`](https://github.com/angular/angular-cli/blob/19.0.2/packages/angular_devkit/build_angular/src/builders/karma).

This is a bit more difficult to solve. As **there is no Karma plugin yet for Knip**. So maybe a Karma plugin would first be needed to pass the configuration to it.

Then, the test running is configured by a mix of the builder options and Karma configuration.

Builder options configure many things as can be seen in the [options schema][karma-builder-options-schema]. Some of those are specified by default when creating the app in `angular.json`. Checkout [the `angular.json` file](./angular.json) to see which ones. Other have some defaults as can be seen in the [options schema][karma-builder-options-schema]. So to be correct, builder options in `angular.json` should be merged with defaults.

Options in there can be grouped in these main categories:

##### App build options

How the app should be built for testing purposes. So it includes many options available in `build` targets. Up until v19 app was built using the traditional, Webpack-based `browser` builder. However, [in v19 there is developer preview support for building with the newer ESBuild-based `application` / `browser-esbuild` builder][v19-test-tooling-state]. Via `builderMode` option

Specifically here are those options and how they could be used for Knip:

- `main`: to add as non-production entry point (as it's a testing entry point)
- `tsConfig`: is set to `tsconfig.spec.json` by default as you can see [in  `angular.json`](./angular.json). To add to Typescript plugin config.
- `polyfills`: to add as non-production entry if points to a local file, or as used dependency.
- `assets`
- `scripts`: same as `polyfills`.
- `styles`: N/A
- `inlineStyleLanguage`: N/A
- `stylePreprocessorOptions`: N/A
- `sourceMap`: N/A
- `progress`: N/A
- `watch`: N/A
- `poll`: N/A
- `preserveSymlinks`: N/A
- `browsers`: N/A
- `fileReplacements`: to add as non-production entry points
- `builderMode`: N/A
- `webWorkerTsConfig`

Taking into account that entry points that are considered production ones, if added because found here, they should not change to be considered non-production ones.

##### Test files to include exclude

Two options for that:

- **`include`**: defines the test files to run. Defaults to `**/*.spec.ts` as can be seen in [options schema][karma-builder-options-schema]. They should be added as non-production entry files. Some quirks:
  - If a directory is specified, it means all `.spec.@(ts|tsx)` inside it will be included. More details [in `findTests`](https://github.com/angular/angular-cli/blob/19.0.2/packages/angular_devkit/build_angular/src/builders/karma/find-tests.ts#L14) and [in `findMatchingTests`](https://github.com/angular/angular-cli/blob/19.0.2/packages/angular_devkit/build_angular/src/builders/karma/find-tests.ts#L47) functions of the builder's implementation.
- **`exclude`**: files to exclude for test running. For more details, check out the functions mentioned above. They should be taken into account alongst with `include` to not add those as entry files.

With that, we'd remove the `app.component.spec.ts` unused file. In a bigger app, there will be lots of them.

##### Coverage options

Specifically:

- `codeCoverage`: whether to generate a code coverage report or not. N/A to knip.
- `codeCoverageExclude`: files to exclude from coverage measure. N/A to knip AFAIK.

Those options may be overridden if specified in Karma configuration. However, given they're not interesting for Knip, nothing to take into account.

##### Karma configuration

Finally, there's the `karmaConfig` option that allows to specify the [Karma configuration][karma-config-file] to use. By default, it's unspecified and the [default Karma configuration hardcoded in the builder][karma-builder-default-config] will be used. However, one can be specified. Which may have been generated [with `ng g config`](https://angular.dev/cli/generate/config) command[^1]

With that, we could get rid of:

- **Unused dependencies**: [default Karma's builder options][karma-builder-default-config] includes many `devDependencies` listed as unused in regular mode. Specifically `karma-jasmine`, `karma-chrome-launcher`, `karma-jasmine-html-reporter` and `karma-coverage`. So the only unused dependency not tracked would be `jasmine-core`. But that one could be hardcoded.

[karma-builder-options-schema]: https://github.com/angular/angular-cli/blob/19.0.2/packages/angular_devkit/build_angular/src/builders/karma/schema.json

[karma-builder-default-config]: https://github.com/angular/angular-cli/blob/19.0.2/packages/angular_devkit/build_angular/src/builders/karma/index.ts#L103

[v19-test-tooling-state]: https://blog.angular.dev/meet-angular-v19-7b29dfd05b84#:~:text=State%20of%20testing%20tooling

[karma-config-file]: https://karma-runner.github.io/6.4/config/configuration-file.html

Finally, as side note, a Jest builder is available too. But [it's still experimental][v19-test-tooling-state]. So it's not the default yet.

##### Plan

So with all this information, the way to go with this could be:

1. **Karma plugin for Knip**
1. Enable if `karma` listed as development dependency.
1. Parse [Karma config file][karma-config-file] if exists.
1. [Test files](https://karma-runner.github.io/6.4/config/files.html): add them as non-production entry points. No defaults for that, it's a mandatory option.
1. [Plugins](https://karma-runner.github.io/6.4/config/configuration-file.html#plugins): add dependencies listed in `plugins` configuration as used, non-production dependencies. Default is all `karma-*` dependencies.
1. **Angular to enable Karma plugin**
1. **Resolve Angular options into a Karma configuration**
1. Files: from builders options and their defaults.
1. Plugins: from [hardcoded default Karma configuration][karma-builder-default-config] if no `karmaConfig` is specified
1. **Send Angular resolved configuration to existing Karma configuration**. Plugins and files. With [Knip's `toConfig`](https://github.com/webpro-nl/knip/blob/5.38.3/packages/knip/src/util/input.ts#49)
1. **Add files used Angular build options in test builder**. Seen in the [app build options](#app-build-options) above.

Plugin will be needed first before then passing configurations to it. However, using the files in Angular build options for the test target is something that could be done already. However, that could come later as it's not something required hence many users may not use those.

#### Others

To end up with a freshly generated app having nothing unused would require also fixing:

##### Unused dependencies

TL;DR: they're properly being reported as unused by default. As they're actually unused.

- `@angular/platform-browser-dynamic`: makes sense to report as unused, as it's not used by default when creating standalone apps ([default since v17](https://blog.angular.dev/introducing-angular-v17-4d7033312e4b#:~:text=Standalone%20APIs%20from%20the%20start)). It [is used](https://github.com/angular/angular-cli/blob/19.0.2/packages/schematics/angular/application/files/standalone-files/src/main.ts.template) for module-based apps. Though it's added there in case you need to. [Seems it allows to run apps that require the JIT compiler on the client](https://angular.dev/reference/configs/npm-packages#default-dependencies:~:text=Includes%20providers%20and%20methods%20to%20compile%20and%20run%20the%20application%20on%20the%20client%20using%20the%20JIT%20compiler.). Can't say more as haven't dealt with it much.
- `@angular/compiler`: same as above. Seems if not using JIT on the client, not needed, could be a development dependency. Can't say much, haven't dealt with it.
- `@angular/forms`: if not using forms, it's correctly to appear as unused, as it's not used. But Angular CLI adds it there by default so you have it there when you want to use it. Users could choose to ignore that one or uninstall it if unused.

##### Unlisted binaries

`ng` appears as unlisted binary in production mode. This is because `start` run script uses `ng serve`. However, Angular CLI is not listed as a production dependency. Maybe that could be ignored by default for Angular projects?

##### Environment files

As seen in [app build options](#app-build-options), when an app has environment files (see [docs](https://angular.dev/tools/cli/environments#configure-environment-specific-defaults)) they should be included too. They aren't included when specified as `build` options either.

##### Scripts and polyfills

As seen in [app build options](#app-build-options), scripts and polyfill files to use in the app can be specified in there. They should be taken into account. Both the ones in `build` and `test` target. Those in `build` as production entries. Those in `test` as non-production entries.

### Tasks

Sorted by high impact, low complexity first

| Status | PR | Name                                             | Complexity | Impact |
|:------:|:--:|:-------------------------------------------------|:----------:|:------:|
|  [ ]   |    | [SSR fixes](#server-side-rendering-ssr)          |     🟢     |   ⏫    | 
|  [ ]   |    | [Environment files](#environment-files)          |     🟡     |   ⬆️   |
|  [ ]   |    | [Karma plugin](#plan)                            |     🔴     |   ⏫    |
|  [ ]   |    | [Angular options to Karma plugin](#plan)         |     🟡     |   ⏫    |
|  [ ]   |    | [Scripts build option](#scripts-and-polyfills)   |     🟢     |   ⬆️   |
|  [ ]   |    | [Polyfills build option](#scripts-and-polyfills) |     🟢     |   ⬆️   |
|  [ ]   |    | [Test build options: `main` / `tsConfig`](#plan) |     🟢     |   ⬇️   |

Complexity means subjective implementation complexity / work / effort.
Impact:
 - ⏫ **very high**: affects freshly baked apps by default
 - ⬆️ **high**: affects apps using subjectively usual custom configurations
 - ⬇️ **low**: affects apps with subjectively unusual custom configurations

## Generation

Command used to generate this project:

```shell
pnpm dlx @angular/cli@19.0.2 new a19-knip \
  --package-manager=pnpm \
  --style=css \
  --ssr --server-routing
```

## Development server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open your browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever you modify any of the source files.

## Code scaffolding

Angular CLI includes powerful code scaffolding tools. To generate a new component, run:

```bash
ng generate component component-name
```

For a complete list of available schematics (such as `components`, `directives`, or `pipes`), run:

```bash
ng generate --help
```

## Building

To build the project run:

```bash
ng build
```

This will compile your project and store the build artifacts in the `dist/` directory. By default, the production build optimizes your application for performance and speed.

## Running unit tests

To execute unit tests with the [Karma](https://karma-runner.github.io) test runner, use the following command:

```bash
ng test
```

## Running end-to-end tests

For end-to-end (e2e) testing, run:

```bash
ng e2e
```

Angular CLI does not come with an end-to-end testing framework by default. You can choose one that suits your needs.

## Additional Resources

For more information on using the Angular CLI, including detailed command references, visit the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli) page.

[^1]: [`karma.conf.js` template](https://github.com/angular/angular-cli/blob/19.0.2/packages/schematics/angular/config/files/karma.conf.js.template)
