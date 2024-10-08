---
title: Version 3 - What's new?
description: Version 3 introduces many new features to the Angular Dynamic Hooks library, such as loading components into existing HTML elements and standalone mode.
---

# Version 3 - What's new?

![An festive image celebrating the release of version 3 of the Angular Dynamic Hooks library](https://github.com/user-attachments/assets/21377754-7f2e-4d54-92bd-e23ba45dbf39)

Version 3 marks a significant overhaul and modernization of the Angular Dynamic Hooks library. It is now fully up-to-date with **Angular 17+**, easier to use and has several completely new features.

**The good news**: Users of version 2 should only minimally need to adjust their code for the upgrade to version 3. Nevertheless, there are some breaking changes to look out for (see below)

## New features

### Standalone components

The whole library was converted to the more modern standalone component structure. This makes it future-proof and allows Angular to further optimize package sizes and bundling logic, such as for <a href="https://angular.dev/guide/defer#which-dependencies-are-defer-loadable" target="_blank">deferred views</a>.

### Less boilerplate

With version 3, configuring global settings/providers has become optional. To get started quickly, the you can pass the content and an array of parsers on each usage of the library individually - nothing else needed.

In addition, you can now use the component classes directly as parsers, as simple as:

```ts
parsers = [ExampleComponent];
```

See the new the [Quickstart example]({{ "guide/quickstart" | relative_url }}) for the full code.

### SSR-Compatibility

The core functionality of the library has been adjusted to work with Server-Side-Rendering under Angular 17+ out of the box. You can however still [implement your own PlatformService]({{ "guide/configuration#alternative-platforms" | relative_url }}), if you have a more specific use-case.

### New: HTML as content

Previously, you could only pass strings as the `content` input to the library. With v3, you can now also submit HTML elements directly - even `document.body`! For example like this:

```ts
dynamicHooksService.parse(document.body, parsers).subscribe(result => {
  // Do whatever
});
```

The library will look through all the child elements, search for hooks/selectors and automatically load components into the current browser content.

### New: Element hooks

Related to that, you can now look for **element hooks** in addition to **text hooks**. Element hooks find and load components into HTML elements (instead of replacing text patterns), so custom parsers can use [simple browser methods]({{ "guide/parsers#findhookelements" | relative_url }}) to find them (`querySelectorAll`, etc.).

The standard `SelectorHookParser` now also uses the new element hooks under the mood. This has the benefit that it is now able to use proper CSS selectors (`.myComponent`) for finding hooks instead of just tag names like before.

### New: Standalone mode

With the new [standalone mode]({{ "guide/standalone-mode" | relative_url }}), you can use the library **outside of an Angular app**. This means, you can load fully-functional Angular components in non-Angular contexts - for example to add "Angular widgets" to HTML generated by a CMS, static page generators, etc. 

All you need is your content and a list of parsers, as always:

```ts
import { parse } from 'ngx-dynamic-hooks';
...

parse(content, parsers).then(result => {
  // Do whatever
});
```

### New: Single component

In case you just need to load a single dynamic component in your template, a new [DynamicSingleComponent]( {{ "guide/single-component" | relative_url }}) was added to make this process as easy as possible.

## Breaking changes

### DynamicHooksModule

As the library has transitioned to standalone components, trying to use `DynamicHooksModule.forRoot()` will throw an error. Instead, use the new <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> function to create the global settings. 

```ts
import { ApplicationConfig } from '@angular/core';
import { provideDynamicHooks } from 'ngx-dynamic-hooks';

export const appConfig: ApplicationConfig = {
  providers: [
    ..
    provideDynamicHooks({
      parsers: [ ... ],
      options: { ... }
    })
  ]
};
```

Additionally, the properties of the settings object have also been simplified:

```ts
interface DynamicHooksSettings {
  parsers?: HookParserEntry[];
  options?: ParseOptions;
  inheritance?: DynamicHooksInheritance;
}
```

Note that using <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> to create global settings is now completely optional. So you can skip this step completely if you want and instead pass the parsers individually to each <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>.

### DynamicHooksComponent

To use `<ngx-dynamic-hooks>` in your templates, you will now have to import the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>. Simply put it into the `imports` field of the component/module that needs it:

```ts
@Component({
  ...
  imports: [DynamicHooksComponent]
})
export class YourComponent {
...
}
```

### Child modules

If you were previously using `DynamicHooksModule.forChild()`, you can now simply call <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/dynamicHooksProviders.ts" target="_blank">`provideDynamicHooks`</a> again in your child providers to register the child settings. You can do this as deeply nested as you want.

Also, note that `DynamicHooksInheritance.LINEAR` is now the default option (instead of `ALL`) as its more in line with angular's default behaviour.

### Switch to HTML-based parsing

In v2 of the library, the standard parser to find hooks used regular expressions to find them in the content. In v3, this was replaced by HTML/DOM-based parsing to improve robustness and to make it easier to write custom hooks.

If you encounter any issues with this switch and configurations that previously worked fine now throw errors, you can return to regex-based parsing by enabling it as an option in your [SelectorHookParserConfig]({{ "guide/parsers#selectorhookparserconfig" | relative_url}}), like this:

```ts
const parsers: HookParserEntry[] = [{
  component: ExampleComponent,
  parseWithRegex: true
}];
```

### Renamings

Several classes, interfaces, methods etc. were renamed to better reflect their new roles. 

- `OutletComponent` is now <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/components/dynamicHooksComponent.ts" target="_blank">`DynamicHooksComponent`</a>
- `OutletService` is now <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/dynamicHooksService.ts" target="_blank">`DynamicHooksService`</a>. Also, the order of the parameters for the `parse` method [has changed]({{ "guide/how-to-use#load-components-via-service" | relative_url }}).
- `OutletParseResult` is now <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L224" target="_blank">`ParseResult`</a> and returns more properties
- `OutletOptions` is now <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/settings/options.ts" target="_blank">`ParseOptions`</a>
- `HookFinder.findStandaloneHooks` is now `HookFinder.findSingletagHooks`. You can now also just use `HookFinder.find` for both singletag or enclosing hooks.

## Minor changes

- Traditional text hook parsers from version 2 remain mostly unchanged - with one exception: They will now use `<dynamic-component-anchor>`-elements for components by default. If you want to continue using the component selector, you must specify it manually via the [`hostElementTag` property]({{ "guide/parsers#loadcomponent" | relative_url }}).
- `DynamicContentChild.componentSelector` field has been removed. you can mostly accomplish the same via `DynamicContentChild.componentRef.location.nativeElement.tagName`.