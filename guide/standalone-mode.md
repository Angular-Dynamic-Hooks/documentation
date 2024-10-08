---
title: Standalone mode
description: With standalone mode, you can mount Angular components directly into HTML that can come from anywhere - without the need for an Angular app.
---

<div class="page-title">
  <img class="page-title-icon" src="{{ "/assets/images/icons/brackets.svg"| relative_url }}" alt="An icon showing a couple of brackets">
  <h1 class="page-title-text">Standalone mode</h1>
</div>

Angular Dynamic Hooks can be used in standalone mode, allowing you to load Angular components independently **without a central Angular app**.

This is ideal for mounting Angular components as "frontend widgets" onto HTML that can come from anywhere, such as a CMS like Wordpress, a dedicated backend framework like Laravel or even just static HTML pages.

{% include docs/notice.html content='
  <h4>Fun fact</h4>
  <p>You are seeing standalone mode in action right now! Though this documentation consists of prerendered HTML files, all interactive elements are actually Angular components! Like this one:</p>  
  <app-example message="This is an Angular component!"></app-example>
' %}

## Getting started

To use standalone mode, simply import the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">`parse`</a> function from the library. It is the equivalent of [DynamicHooksService.parse]({{ "guide/how-to-use#load-components-via-service" | relative_url }}), just with the difference that <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">`parse`</a> is framework-agnostic and can be called from anywhere.

At its most basic, you only need to pass the **content** as well as a list of **parsers**. The [starter example]({{ "/guide/how-to-use#parsing-content" | relative_url }}) would then look like this in standalone mode:

```ts
import { parse } from 'ngx-dynamic-hooks';
import { ExampleComponent } from 'somewhere';

const content = 'Load a component here: <app-example></app-example>';
const parsers = [ExampleComponent];

parse(content, parsers).then(result => {
  // Do whatever
});
```

The full list of parameters <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">can be found here</a>.

Often, you may want to parse the whole page for components. In such cases, you can simply pass `document.body` as content: 

```ts
parse(document.body, parsers)
```

Angular components will then be loaded into all hooks/selectors found anywhere in the browser.

## Adding providers

You can **optionally** specify a list of providers for the loaded components to use, just like in a normal Angular app. For this, import the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L50" target="_blank">`createProviders`</a> function.

It will create a scope with its own internal <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">`parse`</a> method that makes use of the specified providers. A simple example could look like this:

```ts 
import { provideHttpClient } from '@angular/common/http';
import { createProviders } from 'ngx-dynamic-hooks';
import { ExampleComponent } from 'elsewhere';

const scope = createProviders([
  provideHttpClient(),
  // Other providers...
]);

scope.parse(document.body, [ExampleComponent]).then(result => {
  // Do whatever
});
```

Scopes can also inherit providers from each other:

```ts
const parentScope = createProviders([...]);
const childScope = createProviders([...], parentScope);
```

{% include docs/notice.html content='
  <p><b>Tip</b>: Services decorated with <a href="https://angular.dev/guide/di/creating-injectable-service#creating-an-injectable-service" target="_blank"><code>@Injectable(providedIn: "root")</code></a> work without explicitly declaring them as providers and do not need a scope.</p>  
' %}

## Building the components

In terms of writing code, we are already done. In order to compile Angular components however, special build tools are required. This is typically fully handled by the Angular CLI and its `ng build` command.

With standalone mode, we have two options: 

- We can simply also use `ng build` (**recommended**).
- If you have a specialized Webpack build pipeline, we can add Angular plugins to Webpack in order to compile our code **without** the Angular CLI.


### a) Build with the CLI

Using the Angular CLI is the recommended method as it is easy and uses the official build tools.

All that is needed is the <a href="https://angular.dev/tools/cli/setup-local" target="_blank">Angular CLI</a>, an **angular.json** file in your project directory and of course Angular itself as a dependency in your package.json file.

**Tip:** As a starting point, you can simply copy the <a href="https://github.com/MTobisch/Angular-Dynamic-Hooks-Example-Standalone-CLI/blob/main/angular.json" target="_blank">angular.json</a> and <a href="https://github.com/MTobisch/Angular-Dynamic-Hooks-Example-Standalone-CLI/blob/main/package.json" target="_blank">package.json</a> files from the Stackblitz example below. Alternatively, you can always just call `ng new` to create a fresh Angular project somewhere and copy `angular.json` and `package.json` from there.

Make sure all dependencies are installed (including **Angular Dynamic Hooks**) and that in `angular.json` under `project.PROJECTNAME.architect.build.options`:

- `browser` points to your main entry point file.
- `output` points to where you want to build it to. 
- `index` points to a <a href="https://github.com/MTobisch/Angular-Dynamic-Hooks-Example-Standalone-CLI/blob/0989271a5593611006687886c5abd53f6f6fd480/src/index.html" target="_blank">minimal index.html file</a> (breaks otherwise).

Then call `ng build` to build the finished JS files!

<app-stackblitz
  url="https://stackblitz.com/edit/ngx-dynamic-hooks-v3-standalone-cli" 
  fileQueryParam="file=www%2Findex.html"
  image="{{ "/assets/images/stackblitz/standalone_cli.jpg" | relative_url }}"
></app-stackblitz>

{% include docs/notice.html content='
  <h4>Bundled or separate file?</h4>
  <p>If you have unrelated TS/JS code that should be bundled as well, you could simply point <code>ng build</code> to a common entry point file (like <code>main.ts</code>).</p>
  <p>Alternatively, you could compile the standalone mode logic with the Angular components separately into their own file and then include that file independently in the browser (via a script tag or otherwise) when you want to load the Angular components - a bit like Angular Elements.</p>
' %}

### b) Build with Webpack

When you have an existing Webpack-based build pipeline and only wish to incorporate loading Angular components into that, it can make sense to incorporate the Angular build tools into Webpack rather than using the CLI. 

**Some special notes:**

- You will have to adjust `webpack.config.js` to support Angular component compilation (see example below for a working config).
- The bundle size will be larger as with a CLI-compiled Angular app, as a manual Webpack config lacks some of the magic the Angular compiler internally uses.
- If you use it, you will have to import `zone.js` at the top of your entry file.

That said, the rest of the code can be identical as when using the CLI. Here is the same example as before, just with Webpack-compilation this time:

<app-stackblitz
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Standalone-Webpack" 
  fileQueryParam="file=www%2Findex.html"
  image="{{ "/assets/images/stackblitz/standalone_webpack.jpg" | relative_url }}"
></app-stackblitz>

## Trivia

### Using Inputs/outputs

When using standalone mode and you're loading Angular components straight into normal HTML elements, you might be wondering how to pass and receive data from the components. The answer is: Just like in templates! Simply use Angular-style input/output attributes:

```html
<app-example somePlainInput="Hello!" [someInput]="context.someValue" (someOutput)="context.someFunction($event)"></app-example>
```

For more info this, see [the dedicated page about how to use inputs/outputs]({{ "guide/dynamic-component-features#inputs" | relative_url }}).

**Tip: DOM events for outputs** 

In standalone mode, the `triggerDOMEvents` option of the [ParseOptions]({{ "guide/configuration#parseoptions" | relative_url }}) is true by default. This means that alternatively to subscribing to outputs via element attributes, you can also subscribe to them as DOM events triggered from the host element. 

For example, we could also subscribe to the `someOutput` output of the example component above like this:

```ts
  // Get host element
  let componentHostElement = document.querySelector('app-example');
  // Register to output
  componentHostElement.addEventListener('someOutput', value => {
    // Do whatever
  });
```

### React to DOM changes

It is often a good idea to automatically parse the content again when some of its elements change, so the corresponding components are loaded immediately.

For this purpose, you can use the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standaloneHelper.ts#L11" target="_blank">`observeElement`</a> function. It automatically picks up on DOM changes and runs a callback function whenever new elements are added to your content. Here is a simple usage example:

```ts
import { parse, observeElement } from 'ngx-dynamic-hooks';
...

observeElement(document.body, element => {
  parse(element, parsers);
});
```

### Trigger from other scripts

If you want to trigger <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">`parse`</a> manually, but need to do so from another script, using custom browser events for communication is a good solution. For example, add the following to your <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L158" target="_blank">`parse`</a> script:

```ts
  document.addEventListener('parseHooks', event => {
    parse((event as CustomEvent).detail, parsers)
  });
```

You can then dispatch events from anywhere else in the browser to trigger the listener: 

```ts
  document.dispatchEvent(new CustomEvent('parseHooks', { detail: theContent}));
```

### Force service creation

In Angular, services are only constructed when they are requested by a component. This means that if a component does not appear on a hypothetical page of your website, any services that are only injected by that component would also not be created. 

This can be a problem if you have a service that takes care of some global tasks and should always work.

To prevent this from happening, you can use the `APP_INITIALIZER` token provided by Angular. Simply "subscribe" to the the `APP_INITIALIZER` token in the providers given to <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/standalone.ts#L50" target="_blank">`createProviders`</a> and your service will always be constructed when calling `scope.parse`, irrespective if it finds anything.

```ts
  import { APP_INITIALIZER } from '@angular/core';
  ...

  const scope = createProviders([
    {
      provide: APP_INITIALIZER,
      useFactory: () => () => {},
      multi: true,
      deps: [MyCustomService]
    }
  ]);
```

### About Angular elements

Standalone mode fills a similar role to <a href="https://angular.dev/guide/elements" target="_blank">Angular elements</a>: Loading proper Angular components into framework-agnostic HTML that can come from anywhere. 

However, Angular Dynamic Hooks offers several advantages that go beyond what Angular elements is capable of, such as:

- **Anything can be a hook** - not just custom elements. You can load components into any standard HTML element you want and even replace text with components.
- You can easily **lazy-load** components only when they appear on the page.
- Inputs are automatically parsed into their data types, rather than leaving them as strings.

See the full comparison of this library with Angular Elements [on the Trivia page]({{ 'guide/trivia#angular-elements' | relative_url }}).