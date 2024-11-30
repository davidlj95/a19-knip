# A19Knip

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 19.0.2.

Purpose of is to improve [`knip`](https://knip.dev/) defaults for Angular projects. So that it works nicely out-of-the-box.


## Knip setup
`knip` was added following the [getting started guide](https://knip.dev/overview/getting-started).

```shell
pnpm create @knip/config
```

> [`@knip/create-config`](https://www.npmjs.com/package/@knip/create-config) version `1.0.3`
>   
> [`knip`](https://www.npmjs.com/package/knip) version `5.38.3`

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
