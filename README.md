# Angular (19+) `ZoneLess`, `stand-alone`, Agnostic `Web Components` and `Signals`

> Web components are best when we need true reusability, encapsulation, and framework-agnostic UI.

## Goal

- Setup zoneless Angular using standalone components, web components, and signals to share values across components.
- Use injectCdBlink to visualize change detection cycles. Whenever a signal (tracked by an effect in a component) changes, Angular will run change detection for that component, and the DOM for that component will blink (not the whole App). This helps us see, in real time, which components are being checked as we interact with the UI (e.g., clicking increment buttons, changing inputs).

## Demo

![Demo](./public/assets/image.png)


### Results

- We see a blink on load because CD runs once at startup.
- The blink effect will only trigger when the signal(s) read inside the effect change.
- This gives us fine-grained control over which state changes cause visual feedback in each component.

---

## Setup

```js
// Initial project creation with zoneless flag
npx -p @angular/cli@latest ng new [project_name] --experimental-zoneless

// Remove zone.js dependency
npm uninstall zone.js
```

```js
// app.config.ts
import { ApplicationConfig, provideExperimentalZonelessChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideExperimentalZonelessChangeDetection(), // Enables change detection without Zone.js
    provideRouter(routes),  // Provides router configuration
    provideHttpClient()     // Provides HTTP client
  ]
};
```

---

## Signals

> The paradigm, behind Signals, is very similar to RxJS, but a lot simpler. It is possible to link variables together, so that when one is updated, the other updates automatically. It is also possible to use ChangeDetectionStrategy.OnPush and signal changes will be detected automatically, without having to use this.cd.markForCheck()

---

## ChangeDetectionStrategy.OnPush strategy

Without Zone.js, change detection works differently:
Now, instead of monitoring every change during each cycle, Angular checks only when:

- Changes are only detected when component inputs change
- Event emissions trigger change detection (e.g., a click)
- No need for manual markForCheck()

## Managing Changes Without Zone.js

In a zoneless application, we need to be more "explicit" about change detection:

- Changes are automatically detected for input properties

- `Forcing Detection`: For other changes, we'll need to manually trigger change detection using ChangeDetectorRef

```js
private cdRef inject(ChangeDetectorRef) {}
triggerChangeDetection() {
  this.cdRef.detectChanges();
}
```

## Web-Components Support

### Framework Agnostic
- Web components are a browser standard. They work in any framework (Angular, React, Vue, Svelte, etc.) or even plain HTML/JS.
- we can build a component once and use it everywhere.
- Angular Elements lets we export Angular components as web components, so we get the power of Angular with the portability of web components.
- Web components have some limitations (e.g., dependency injection, advanced data binding, SSR) compared to framework-native components.

### Lets transform Angular app into a Web Component:

```js
// Install @angular/elements package
npm install @angular/elements
```

### Create a Wrapper Component
To create a Web Component, a wrapper component is required to encapsulate the entire application. This component will not only export Angular logic as a Web Component but is also essential for preserving router functionality, which is managed in the AppComponent.

`AppComponent will maintains control over routing and application logic`, while the `WrapperComponent becomes the Web Component that we export`


## Transform Angular Components into Web Components

> bootstrapApplication, which is the only supported way to provide all required Angular services (including for zoneless and router) for a standalone web component.

```js
// main.ts
bootstrapApplication(WrapperComponent, appConfig).then(appRef => {
  const wrapper = createCustomElement(WrapperComponent, { injector: appRef.injector });
  customElements.define('my-web-component', wrapper);
});
```

Now, application is encapsulated within the wrapper and exported as a Web Component using @angular/element


## Change-Detection Works with Signals in Zoneless Angular

✅  Signals are reactive primitives. When we update a signal (ej. counter.set(counter() + 1)), `Angular will automatically trigger change-detection for any component that reads that signal` in its template or in a computed/effect.
✅  `Zoneless mode means Angular does NOT monkey-patch async APIs` (like setTimeout, XHR, etc.) to auto-trigger change detection. `Instead, Angular only runs change detection when`:
- An `input changes`
- An `event` handler (ej. click()) runs
- `Manually trigger` it (e.g., with ChangeDetectorRef.detectChanges())


---

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 19.2.9.

## Development server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open wer browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever we modify any of the source files.

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

This will compile wer project and store the build artifacts in the `dist/` directory. By default, the production build optimizes wer application for performance and speed.

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

Angular CLI does not come with an end-to-end testing framework by default. we can choose one that suits wer needs.

## Additional Resources

For more information on using the Angular CLI, including detailed command references, visit the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli) page.

---

### :100: <i>Thanks!</i>
#### Now, don't be an stranger. Let's stay in touch!

<a href="https://github.com/leolanese" target="_blank" rel="noopener noreferrer">
  <img src="https://scastiel.dev/api/image/leolanese?dark&removeLink" alt="leolanese's GitHub image" width="600" height="314" />
</a>

##### :radio_button: Linkedin: <a href="https://www.linkedin.com/in/leolanese/" target="_blank">LeoLanese</a>
##### :radio_button: Twitter: <a href="https://twitter.com/LeoLanese" target="_blank">@LeoLanese</a>
##### :radio_button: DEV.to: <a href="https://www.dev.to/leolanese" target="_blank">Blog</a>
##### :radio_button: Questions / Suggestion / Recommendation: developer@leolanese.com