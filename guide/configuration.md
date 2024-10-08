---
title: Configuration
description: Lists all the options available to configure how Angular components are parsed and loaded from dynamic content by Angular Dynamic Hooks.
---

<div class="page-title">
  <img class="page-title-icon" src="{{ "/assets/images/icons/cog.svg"| relative_url }}" alt="An icon of a cog">
  <h1 class="page-title-text">Configuration</h1>
</div>

## Global settings

You can optionally set up global parsers and options shared between all <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>s in your app by using the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> function in your app providers.

```ts
import { ApplicationConfig } from '@angular/core';
import { provideDynamicHooks } from 'ngx-dynamic-hooks';

export const appConfig: ApplicationConfig = {
  providers: [
    provideDynamicHooks(...)
  ]
};
```

It accepts a <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/settings/settings.ts" target="_blank">`DynamicHooksSettings`</a>-object with the following properties:

Name | Type | Description
--- | --- | ---
`parsers` | <a href="{{ '/guide/parsers' | relative_url }}">HookParserEntry</a>`[]` | An list of hook parsers to provide to all <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>s
`options` | <a href="{{ '/guide/configuration#parseoptions' | relative_url }}">ParseOptions</a> | An options object to provide to all <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>s
`inheritance` | `number` | An enum option from <a href="{{ '/guide/configuration#child-settings' | relative_url }}">DynamicHooksInheritance</a>

You can [see an example here]({{ "guide/how-to-use#global-settings" | relative_url }}). 

## Child settings

You can provide additional parsers and options simply by calling <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> again in the `providers`-fields of child injector contexts, such on <a href="https://angular.dev/api/router/Route#providers" target="_blank">specific routes</a> or even directly on <a href="https://angular.dev/api/core/Component#providers" target="_blank">components</a>. 

The child settings will be merged with other provided settings according to the value of the optional `inheritance` property in the [DynamicHooksSettings]({{ "guide/configuration#global-settings" | relative_url }}) object. It accepts a value from the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/settings/settings.ts" target="_blank">`DynamicHooksInheritance`</a> enum, which are as follows:

1. `DynamicHooksInheritance.Linear`: (Default) Only merges with settings from direct ancestor injectors (such a father and grandfather injectors, but not "uncle" injectors).
2. `DynamicHooksInheritance.All`: Merges with settings from all injectors in the app.
3. `DynamicHooksInheritance.None`: Does not merge at all. Injector only uses own settings.

An example for using child settings might then look like this:

```ts
providers: [
  ...
  provideDynamicHooks({
    parsers: [ ... ],
    options: { ... },
    inheritance: DynamicHooksInheritance.None
  })
]
```

## DynamicHooksComponent

These are all of the inputs you can pass to each <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> (`<ngx-dynamic-hooks>`) individually:

Input name | Type | Description
--- | --- | ---
`content` | `any` | The content to parse and render. Can be a string or HTML element.
`context` | `any` | An optional object to pass data to the dynamically-loaded components
`globalParsersBlacklist` | `string[]` | An optional list of global parsers to blacklist, identified by their name
`globalParsersWhitelist` | `string[]` | An optional list of global parsers to whitelist, identified by their name
`parsers` | <a href="{{ '/guide/parsers' | relative_url }}">HookParserEntry</a>`[]` | An optional list of hook parsers to use instead of the global parsers
`options` | <a href="{{ '/guide/configuration#parseoptions' | relative_url }}">ParseOptions</a> | An optional options object to use instead of the global options

There is also an output you may subscribe to:

Output name | Type | Description
--- | --- | ---
`componentsLoaded` | `Observable<LoadedComponent[]>` | Will trigger once all components have loaded (including [lazy-loaded ones]({{ "guide/configuration#lazy-loading-components" | relative_url }}))

Each <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L238" target="_blank">`LoadedComponent`</a> from the output represents a dynamically-created component and contains some information you may find interesting:

```ts
interface LoadedComponent {
    hookId: number;                     // The unique hook id
    hookValue: HookValue;               // The hook that was replaced by this component
    hookParser: HookParser;             // The associated parser
    componentRef: ComponentRef<any>;    // The created componentRef
}
```

## ParseOptions

You can define <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/settings/options.ts" target="_blank">`ParseOptions`</a> whenever you parse content - either in the global settings, as an input to each <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> or as a parameter when using the library programmatically.

These options can be used to customize the parsing behaviour:

Option name | Type | Default | Description
--- | --- | --- | ---
`sanitize` | `boolean` | `depends` | Whether to use Angular's `DomSanitizer` to sanitize the content (hooks are unaffected by this). Defaults to `true` if content is a string, `false` if its an HTML element.
`convertHTMLEntities` | `boolean` | `true` | Whether to replace HTML entities like `&amp;` with normal characters.
`fixParagraphTags` | `boolean` | `true` | When using a WYSIWYG-editor, enclosing text hooks may collide with its generated HTML (the `<p>`-tag starting before the hook and the corresponding `</p>`-tag ending inside, and vice versa). This will result in faulty HTML when rendered in a browser. This setting removes these ripped-apart tags.
`updateOnPushOnly` | `boolean` | `false` | Whether to update the bindings of dynamic components only when the context object passed to the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a> changes by reference.
`compareInputsByValue` | `boolean` | `false` | Whether to deeply-compare inputs for dynamic components by their value instead of by their reference on updates.
`compareOutputsByValue` | `boolean` | `false` | Whether to deeply-compare outputs for dynamic components by their value instead of by their reference on updates.
`compareByValueDepth` | `number` | `5` | When comparing by value, how many levels deep to compare them (may impact performance).
`triggerDOMEvents` | `boolean` | `depends` | Whether to emit <a href="https://developer.mozilla.org/en-US/docs/Web/Events/Creating_and_triggering_events" target="_blank">CustomEvents</a> from the component host elements when an output emits. The event name will be the output name. Defaults to true in standalone mode, otherwise false.
`ignoreInputAliases` | `boolean` | `false` | Whether to ignore input aliases like `@Input('someAlias')` in dynamic components and use the actual property names instead.
`ignoreOutputAliases` | `boolean` | `false` | Whether to ignore output aliases like `@Output('someAlias')` in dynamic components and use the actual property names instead.
`acceptInputsForAnyProperty` | `boolean` | `false` | Whether to disregard `@Input()`-decorators completely and allow passing in values to any property in dynamic components. Does not trigger ngOnChanges and can cause issues with change detection in zoneless mode.
`acceptOutputsForAnyObservable` | `boolean` | `false` | Whether to disregard `@Output()`-decorators completely and allow subscribing to any `Observable` in dynamic components.
`logOptions` | `LogOptions` | `{dev: true}` | Accepts a `LogOptions` object to customize when to log text, warnings and errors.

## Lazy-loading components

You can configure components to lazy-load only when its corresponding hook appears in the content. This reduces bundle sizes and saves bandwidth if the hook does not appear at all.

**Using this feature is easy:** Instead of using the component class directly, use a function that returns a promise with the component class instead. This works similar to <a href="https://angular.dev/guide/ngmodules/lazy-loading" target="_blank">lazy-loading routes in Angular</a>.

You can put that function in the `component`-field of a [SelectorHookParserConfig]({{ "guide/parsers#selectorhookparserconfig" | relative_url }}). You also need to manually specify a selector, as it cannot be known before loading the component class, like so:

```ts
import { Component } from '@angular/core';
import { DynamicHooksComponent } from 'ngx-dynamic-hooks';

@Component({
  ...
  imports: [DynamicHooksComponent]
})
export class AppComponent {
  content = 'Load a component here: <app-lazy></app-lazy>';
  parsers = [
    {
      component: () => import('./components/lazyComponent').then(m => m.LazyComponent),
      selector: 'app-lazy'
    }
  ]
}
```

```html
<ngx-dynamic-hooks [content]="content" [parsers]="parsers"></ngx-dynamic-hooks>
```

That's all there is to it! `LazyComponent` will now be lazily-loaded if `<app-lazy>...</app-lazy>` is found in the content.

**Tip:** It you are using a custom parser, you can return the lazy-loading function as part of `loadComponent()` instead.

{% include docs/notice.html content="
  <p>Note that it must be a function that returns a promise, not the promise itself! Otherwise the promise would be executed right where it is defined, which defeats the point of lazy-loading.</p>
" %}

## Alternate platforms

The default implementation of the library should work in both <a href="https://angular.dev/api/platform-browser/bootstrapApplication" target="_blank">browsers</a> as well as during <a href="https://angular.dev/guide/ssr" target="_blank">server-side-rendering</a>. However, there may be more specialized use cases on platforms that are not directly supported.

In such cases, you can create your own <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/platformService.ts" target="_blank">`PlatformService`</a>. The <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/platformService.ts" target="_blank">`PlatformService`</a> is internally used as a layer of abstraction between the library and the platform it runs on. It offers several functions to interact with the platform and handles platform-specific objects (such as `document` and `HTMLElement` in the case of the default <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/platformService.ts" target="_blank">`PlatformService`</a>).

You can implement your own <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/platformService.ts" target="_blank">`PlatformService`</a> by creating a class that follows the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/platformService.ts" target="_blank">PlatformService interface</a> and pass it as the second parameter to <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a>.

**Tip:** You can partially implement as many methods as you need. For all non-implemented methods, the library falls back to the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/platform/defaultPlatformService.ts" target="_blank">default implementation</a>.