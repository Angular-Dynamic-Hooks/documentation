---
title: How to use
description: This page shows the most common ways to utilize the Angular Dynamic Hooks library to load components into dynamic templates or HTML content.
---

<div class="page-title">
  <img class="page-title-icon" src="{{ "/assets/images/icons/book.svg"| relative_url }}" alt="An icon of a book">
  <h1 class="page-title-text">How to use</h1>
</div>

There are two ways to use the library in Angular: In a template via the [DynamicHooksComponent]({{ "guide/configuration#dynamichookscomponent" | relative_url }}) or programmatically via the [DynamicHooksService]({{ "guide/how-to-use#load-components-via-service" | relative_url }}).

## Parsing content

In the [Quickstart example]({{ 'guide/quickstart' | relative_url }}), we have already seen how to use the component in a minimal way. Just import the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> where you need it and pass in your content (the "dynamic template") as well as the components to look for:


```ts
import { Component } from '@angular/core';
import { DynamicHooksComponent } from 'ngx-dynamic-hooks';
import { ExampleComponent } from 'somewhere';

@Component({
  ...
  imports: [DynamicHooksComponent]
})
export class AppComponent {
  content = 'Load a component here: <app-example></app-example>';
  parsers = [ExampleComponent];
}
```

```html
<ngx-dynamic-hooks [content]="content" [parsers]="parsers"></ngx-dynamic-hooks>
```

{% include docs/notice.html content='
  <span>This example uses a string as the content, but you can always also use an actual HTML element. The hooks/components will be loaded in the same way.</span>
' %}

## Using inputs & outputs

You can easily [pass data]({{ 'guide/dynamic-component-features#inputs' | relative_url }}) to your dynamic components, just like in normal Angular templates. Assuming our `ExampleComponent` has an input called `message`, you could use it like so:

```ts
  content = `<app-example [message]="'Hello there!'"></app-example>`;
```

Outputs can be used as in templates as well. Here's an example that calls a function from the context object (see next section):

```ts
  content = `<app-example (myOutput)="context.doSomething()"></app-example>`;
```

## Context and options

There are several more inputs for the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>, most notably **context** to bind data to the loaded components and **options** to configure how the content should be parsed. 

```ts
@Component({
  ...
  imports: [DynamicHooksComponent]
})
export class AppComponent {
  content = 'Load a component here: <app-example [message]="context.someString"></app-example>';
  parsers = [ExampleComponent];
  context = {someString: "Greetings from the context object!"};
}
```

```html
<ngx-dynamic-hooks 
  [content]="content" 
  [parsers]="parsers" 
  [context]="context"
  [options]="{sanitize: false}"
></ngx-dynamic-hooks>
```

In this example, we're passing the value of `context.someString` from the parent component to the `[message]`-input of `ExampleComponent` with the help of the context object. You can put whatever you want into the context object and use it in the inputs/outputs of the dynamic components. [See here]({{ 'guide/dynamic-component-features#context--dependency-injection' | relative_url }}) for detailed info about how to use it.

Also, as we know the content string is safe and does not contain malicous code, we can set the `sanitize`-option to `false` (`true` by default). You can read about all available options on the [configuration page]({{ 'guide/configuration#parseoptions' | relative_url }}).

See it in action in this Stackblitz:

<app-stackblitz
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Context" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/context.jpg" | relative_url }}"
></app-stackblitz>

## Global settings

If you use <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> more than once, it can become tedious to manually pass along the desired parsers and options every time.

Instead, you can also register them [globally]({{ "guide/configuration#global-settings" | relative_url }}) in your app providers with <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a>:

```ts
import { ApplicationConfig } from '@angular/core';
import { provideDynamicHooks } from 'ngx-dynamic-hooks';
import { ExampleComponent } from './components/example/example.component';

export const appConfig: ApplicationConfig = {
  providers: [
    ...
    provideDynamicHooks({
      parsers: [ExampleComponent],
      options: {
        sanitize: false
      }
    })
  ]
};
```

Every <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> will then use them by default (unless locally overwritten). Now we can pass just the `content`-input:

```html
<ngx-dynamic-hooks [content]="content"></ngx-dynamic-hooks>
```

See it in action in this Stackblitz:

<app-stackblitz 
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-GlobalSettings" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/global_settings.jpg" | relative_url }}"
></app-stackblitz>

{% include docs/notice.html content='
  <span>If you are using modules, you can put the call to "provideDynamicHooks" into the "providers" decorator field of your module instead.</span>
' %}

You can also call <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> again in the providers fields of child injectors, such as in lazily-loaded routes. These [child settings]({{ "guide/configuration#child-settings" | relative_url }}) will automatically inherit the root settings according to the Angular injector hierarchy.

## Load by any selector

So far, we've been using the component classes themselves as parsers. This is actually a shorthand, as the library automatically sets up a `SelectorHookParser` for each submitted class. If we want, we can manually configure a `SelectorHookParser` instead for more control. 

Let's say we want to load `ExampleComponent`s by the selector `.myWidget` instead of its default element. This might look like so:

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    ...
    provideDynamicHooks({
      parsers: [
        {
          component: ExampleComponent,
          selector: ".myWidget" // Accepts any CSS selector
        }
      ]
    })
  ]
};
```

Now this content string (or equivalent HTML) would work:

```ts
export class AppComponent {
  content = 'Load a component here: <div class="myWidget"></div>';
}
```

As you can see, we replaced the `ExampleComponent` class in the parsers array with a more explicit configuration object that specifies a `component` and `selector`. This is a [SelectorHookParserConfig]({{ "guide/parsers#selectorhookparserconfig" | relative_url }}) and it offers several options to customize how a component is parsed from the content.

See it in action in this Stackblitz:

<app-stackblitz 
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-SelectorHookParserConfig" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/selectorhookparserconfig.jpg" | relative_url }}"
></app-stackblitz>

## Custom parsers

If you need even more flexiblity (such as replacing pure text with components), you can consider implement your own <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a>!

A <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> is quite simple and just needs three methods: One that tells the library how to find the hooks, one that returns the component class and one that returns the input/output values.

For a full guide with stackblitz examples, see the [Writing your own HookParser]({{ "guide/parsers#writing-your-own-hookparser" | relative_url }}) section.

## Load components via service

You can also parse dynamic content directly in Typescript by injecting the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/dynamicHooksService.ts" target="_blank">`DynamicHooksService`</a> and calling its `parse`-method programmatically. Here is a simple example:

```ts
import { DynamicHooksService } from 'ngx-dynamic-hooks';

...
class AppComponent {
  constructor(private dynamicHooksService: DynamicHooksService) {
    
    const content = 'Load a component here: <app-example></app-example>';
    // const content = document.querySelector('#content'); // Also possible

    dynamicHooksService.parse(content).subscribe(result => {
      // Do whatever with it
    });

  }
}
```

The only required parameter is `content`, but it optionally accepts a lot more:

```ts
  parse(
    content: any = null,
    parsers: HookParserEntry[]|null = null,
    context: any = null,
    options: ParseOptions|null = null,
    globalParsersBlacklist: string[]|null = null,
    globalParsersWhitelist: string[]|null = null,
    targetElement: HTMLElement|null = null,
    targetHookIndex: HookIndex = {},
    environmentInjector: EnvironmentInjector|null = null,
    injector: Injector|null = null
  ): Observable<ParseResult>
```

If this seems familiar, its because most parameters are just [the inputs]({{ "guide/configuration#dynamichookscomponent" | relative_url }}) for the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> component. 

Only some are notable: You can optionally provide a `targetElement` and `targetHookIndex` to fill out for the result. If not, they are automatically created for you. You may also specify custom injectors for the created components. If you don't, the library defaults to the current ones.

The function will return an <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L224" target="_blank">`ParseResult`</a> observable:

```ts
interface ParseResult {
    element: any;                                 // The parsed content element
    hookIndex: HookIndex;                         // An object with the generated hook data
    context: any;                                 // The used context object  for the result
    usedParsers: HookParser[];                    // The used parsers for the result
    usedOptions: ParseOptions;                    // The used options for the result
    usedInjector: Injector;                       // The used injector for the result
    usedEnvironmentInjector: EnvironmentInjector; // The used environment injector for the result
    destroy: () => void;                          // Destroys all loaded components of this result
}
```
`element` is probably the most interesting part here as it contains the finished content with all loaded component elements. <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L12" target="_blank">`hookIndex`</a> might also prove useful, as it is a fairly in-depth data object that holds various tidbits of info concerning the loaded components (as well as the componentRefs). 

See it in action in this Stackblitz:

<app-stackblitz
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Service" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/service.jpg" | relative_url }}"
></app-stackblitz>

{% include docs/notice.html content="
  <h4>About component lifecycles</h4>
  <p>When loading components this way, keep in mind that the submitted content is only parsed once. The inputs of contained components aren't automatically updated.</p>
  <p>Also, make sure to properly destroy the created components when they are no longer needed to prevent memory leaks. You can simply use <code>ParseResult.destroy()</code> or <code>DynamicHooksService.destroy(hookIndex: HookIndex)</code> for this purpose.</p>
" %}