---
title: Parsers
description: Parsers find component hooks in the dynamic content, specify how the component should be loaded and which inputs and outputs to pass along to it.
---

<div class="page-title">
  <img class="page-title-icon" src="{{ "/assets/images/icons/magnifying_glass.svg"| relative_url }}" alt="An icon of a magnifying glass">
  <h1 class="page-title-text">Parsers</h1>
</div>

## Introduction

Components are loaded from [hooks]({{ "guide/#whats-a-hook" | relative_url }}) in the content, but how does the library know what a hook looks like and which component to load for it? This job is accomplished by **Hook parsers**. 

They are what you pass along as the `parsers` input/argument to the library. Each component has one and each must be of the type <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/settings/parserEntry.ts" target="_blank">`HookParserEntry`</a>, which can be either:

1. The component class itself.
2. A <a href="{{ "guide/parsers#selectorhookparserconfig" | relative_url }}">SelectorHookParserConfig</a> object literal.
3. A custom <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> instance.
4. A custom <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> class. If this class is available as a provider/service, it will be injected.

Using the component class is the most straightforward option. It internally sets up a `SelectorHookParser` for you which loads components just like in Angular templates. We have been using it in most simple examples, such as in [Quick Start]({{ "guide/quickstart" | relative_url }}) and most of the [How to use]({{ "guide/how-to-use" | relative_url }}) page.

If you want more control, you can also manually configure a `SelectorHookParser` by passing in a [SelectorHookParserConfig]({{ "guide/parsers#selectorhookparserconfig" | relative_url }}), which provides additional options.

For even more specific use-cases, you may want to write your own HookParser. See the section [Writing your own HookParser]({{ "guide/parsers#writing-your-own-hookparser" | relative_url }}) for more info about that.

## SelectorHookParserConfig

A <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/parsers/selector/selectorHookParserConfig.ts" target="_blank">`SelectorHookParserConfig`</a> is an object literal that can be used in the `parsers` field to create customized `SelectorHookParser` for you (which loads components by their selectors similarly to Angular).

In its simplest form, it just contains the component class like `{component: ExampleComponent}`, but it also accepts additional properties: 

Property | Type | Default | Description
--- | --- | --- | ---
`component` | `ComponentConfig` | - | The component to be used. Can be its class or a function that returns a promise with the class (lazy-loading).
`name` | `string` | - | The name of the parser. Only required if you want to black- or whitelist it.
`selector` | `string` | The component selector | The selector to use to find the hook.
`hostElementTag` | `string` | - | A custom tag to be used for the component host element.
`parseWithRegex` | `boolean` | `false` | Whether to use regex rather than HTML/DOM-based methods to find the hook elements.
`allowSelfClosing` | `boolean` | `true` | Whether to allow using self-closing tags (`<hook/>`). Must use **regex** mode for this (see box below).
`enclosing` | `boolean` | `true` | **Deprecated**: Whether the selector is enclosing (`<hook>...</hook>`) or not (`<hook>`). Use `allowSelfClosing` for a more modern approach.
`bracketStyle` | `{opening: string, closing: string}` | `{opening: '<', closing: '>'}` | The brackets to use for the selector.
`parseInputs` | `boolean` | `true` | Whether to parse inputs into data types or leave them as strings.
`unescapeStrings` | `boolean` | `true` | Whether to remove escaping backslashes from inputs.
`injector` | <a href="https://angular.dev/api/core/Injector" target="_blank">`Injector`</a> | The nearest one | The Injector to create the component with.
`environmentInjector` | <a href="https://angular.dev/api/core/EnvironmentInjector" target="_blank">`EnvironmentInjector`</a> | The nearest one | The EnvironmentInjector to create the component with.
`inputsBlacklist` | `string[]` | `null` | A list of inputs to ignore.
`inputsWhitelist` | `string[]` | `null` | A list of inputs to allow exclusively.
`outputsBlacklist` | `string[]` | `null` | A list of outputs to ignore.
`outputsWhitelist` | `string[]` | `null` | A list of outputs to allow exclusively.
`allowContextInBindings` | `boolean` | `true` | Whether to allow the use of context object variables in inputs and outputs.
`allowContextFunctionCalls` | `boolean` | `true` | Whether to allow calling context object functions in inputs and outputs.

See the [How to use]({{ "guide/how-to-use#load-by-any-selector" | relative_url }}) page for a simple <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/parsers/selector/selectorHookParserConfig.ts" target="_blank">`SelectorHookParserConfig`</a> example.

{% include docs/notice.html content="
  <p>The SelectorHookParser uses HTML/DOM-based methods to find hook elements by default. However, it will switch over to <b>regex-based parsing</b> if either <code>parseWithRegex</code> is true, <code>enclosing</code> is false or you use a custom <code>bracketStyle</code>. This allows the library to also find elements that are technically not valid HTML.</p>
  <p>However, please note that when using regex-based parsing, you cannot use full CSS selectors in the <code>selector</code> field. The selector can then only be the direct tag name, e.g. <code>app-example</code>.</p>
" %}

## Writing your own HookParser

![An example for a custom parser in the Angular Dynamic Hooks library](https://i.imgur.com/9J9t5ze.png)

So far, we have only used the standard `SelectorHookParser`, which is included in this library for convenience and is easy to use if all you need is to load components by their selectors. However, by creating custom parsers, any element or text pattern you want can be replaced by an Angular component.

### What makes a parser

A hook parser is any class that follows the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">HookParser interface</a>, which requires the following:

* An optional `name` property that is used for black/whitelisting the parser.
* `findHooks()` or `findHookElements()` to tell the library where to load the components.
* `loadComponent()` to specify which component class to load.
* `getBindings()` to return the component inputs/outputs.

You only need to implement either `findHooks()` or `findHookElements()`, depending on whether you want to replace text or HTML elements with components. 

The following section explains these main functions in detail. If you would rather see a custom parser in action right away, you can [skip ahead to the examples]({{ "guide/parsers#example-1-minimal" | relative_url }}).

### findHooks()

```ts
  findHooks(content: string, context: any, options: ParseOptions): HookPosition[]
```

Is given a string of content and is expected to return a <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L109" target="_blank">`HookPosition`</a> array from it. Each <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L109" target="_blank">`HookPosition`</a> represents a found text hook and specifies its position within the content string:

```ts
interface HookPosition {
    openingTagStartIndex: number;
    openingTagEndIndex: number;
    closingTagStartIndex?: number;
    closingTagEndIndex?: number;
}
``` 

The opening and closing tags simply refer to the text patterns that signal the start and end of the hook and thereby also define the `<ng-content>` (think `[HOOK_OPENINGTAG]...content...[HOOK_CLOSINGTAG]`). If you are looking for a singletag rather than an enclosing hook (`...[HOOK]....`), you can just omit the two closing tag indexes.

How your hook looks like and how you find these indexes is completely up to you. You may look for them using Regex patterns or any other parsing method. Though, as a word of warning, do not try to parse enclosing hooks with Regex alone. <a href="https://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454" target="_blank">That road leads to madness</a>.

To make your life easier, you can just use the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/utils/hookFinder.ts" target="_blank">`HookFinder`</a> service that comes with this library. Its easy to use and safely finds both singletag and enclosing patterns in a string. You can see it in action in the ["Emoji parser" example]({{ "guide/parsers#example-2-emoji-parser" | relative_url }}).

{% include docs/notice.html content="
  <span><code>findHooks()</code> is only needed if you want to find text hooks. For element hooks, see <code>findHookElements()</code>.</span>
" %}

### findHookElements()

```ts
  findHookElements(contentElement: any, context: any, options: ParseOptions): any[]
```

Is given the main content element and is expected to return an array of child elements that should be used as element hooks.

Finding element hooks is rather easy as you can interact directly with the actual content element. You can typically just do something like this:

```ts
return Array.from(contentElement.querySelectorAll('.myHook'));
```

{% include docs/notice.html content="
  <span><code>findHookElements()</code> is only needed if you want to find element hooks. For text hooks, see <code>findHooks()</code>.</span>
" %}

### loadComponent()

```ts
  loadComponent(hookId: number, hookValue: HookValue, context: any, childNodes: any[], options: ParseOptions): HookComponentData
```

Is given the found hook string or element as <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L119" target="_blank">`HookValue`</a> and is expected to return a <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L129" target="_blank">`HookComponentData`</a> object, which tells the library how to create the component for this hook:

```ts
interface HookComponentData {
    component: ComponentConfig;
    hostElementTag?: string;
    injector?: Injector;
    environmentInjector?: EnvironmentInjector;
    content?: any[][];
}
```

You usually only need to fill out the `component` field, which can be the component class or a <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L157" target="_blank">`LazyLoadComponentConfig`</a> (see [Lazy-loading components]({{ "guide/configuration#lazy-loading-components" | relative_url }})). 

You may optionally also specify a custom host element tag, provide your own injectors or use custom content to replace the existing `<ng-content>` (each entry in the outer array represends a `<ng-content>`-slot and the inner array its content).

### getBindings()

```ts
  getBindings(hookId: number, hookValue: HookValue, context: any, options: ParseOptions): HookBindings;
```

Is given the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L119" target="_blank">`HookValue`</a> and is expected to return a <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L165" target="_blank">`HookBindings`</a> object, which contains all the current inputs and outputs for the component:

```ts
interface HookBindings {
    inputs?: {[key: string]: any};
    outputs?: {[key: string]: (event: any, context: any) => any};
}
```

Both `inputs` and `outputs` must contain an object where each key is the name of the binding and each value what should be used for it. The functions you put in `outputs` will be called when the corresponding @Output() triggers and are automatically given the event object as well as the current context object as parameters. To disallow or ignore inputs/outputs, simply don't include them here.

How you determine the values for the component bindings is - again - completely up to you. You could for example have a look at the <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L119" target="_blank">`HookValue`</a> and read them from the hook itself (like property bindings in selector hooks, e.g. `[input]="'Hello!'`"). You could of course also just pass static values into the component.

{% include docs/notice.html content="
  <h4>Warning</h4>
  <p>Don't use JavaScript's <code>eval()</code> function to parse values from text into code, if you can help it. It can create massive security loopholes. If all you need is a way to safely parse strings into standard JavaScript data types like strings, numbers, arrays, object literals etc., you can simply use the <code>evaluate()</code> method from the <code><a href='https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/services/utils/dataTypeParser.ts' target='_blank'>DataTypeParser</a></code> service that you can also import from this library.</p>
" %}

## Example 1: Minimal

Let's write a a minimal custom <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> for our trusty `ExampleComponent`:

```ts
import { Injectable } from '@angular/core';
import { HookParser, HookValue, HookComponentData, HookBindings } from 'ngx-dynamic-hooks';
import { ExampleComponent } from 'somewhere';

@Injectable({ 
  providedIn: 'root' 
})
export class ExampleParser implements HookParser {

  findHookElements(contentElement: any): any[] {
    // Return the elements to load the component into
    return Array.from(contentElement.querySelectorAll('some-element'));
  }

  loadComponent(): HookComponentData {
    // Return the component class
    return { component: ExampleComponent };
  }

  getBindings(): HookBindings {
    // Return inputs/outputs to set
    return {
      inputs: {
        message: 'Hello there!'
      }
    }
  }
}
```

Now just pass the parser in the `parsers`-field and it will work!

```ts
...
export class AppComponent {
  content = 'Load a component here: <some-element></some-element>';
  parsers = [ExampleParser];
}
```

```html
<ngx-dynamic-hooks [content]="content" [parsers]="parsers"></ngx-dynamic-hooks>
```

See it in action in this Stackblitz:

<app-stackblitz 
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Parsers-Minimal" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/parser_minimal.jpg" | relative_url }}"
></app-stackblitz>

## Example 2: Emoji parser

Let's say we want to replace replace text emoticons (smileys etc.) with an `EmojiComponent` that renders animated emojis for them.

This means that we need to **replace text** instead of HTML elements with components this time and therefore must use `findHooks()` instead of `findHookElements()`.

For the purpose of this example, we have a simple `EmojiComponent` that supports three emojis. It has a `type`-input that determines which one to load (can be either `laugh`, `wow` or `love`). The parser could then look like so:

```ts
import { Injectable } from '@angular/core';
import { HookParser, HookValue, HookComponentData, HookBindings, HookFinder, HookPosition } from 'ngx-dynamic-hooks';
import { EmojiComponent } from './emoji.component';

@Injectable({ 
  providedIn: 'root' 
})
export class EmojiParser implements HookParser {

  constructor(private hookFinder: HookFinder) {}

  findHooks(content: string): HookPosition[] {
    // As an example, this regex finds the emoticons :-D, :-O and :-*
    const emoticonRegex = /(?::-D|:-O|:-\*)/gm;

    // We can use the HookFinder service provided by the library to easily
    // find the HookPositions of any regex in the content string
    return this.hookFinder.find(content, emoticonRegex);
  }

  loadComponent(): HookComponentData {
    return { component: EmojiComponent };
  }

  getBindings(hookId: number, hookValue: HookValue): HookBindings {
    // Lets see what kind of emoticon this hook is and assign a fitting emoji
    let emojiType: string;
    switch (hookValue.openingTag) {
      case ':-D': emojiType = 'laugh'; break;
      case ':-O': emojiType = 'wow'; break;
      case ':-*': emojiType = 'love'; break;
    }

    // Set the 'type'-input in the EmojiComponent correspondingly
    return {
      inputs: {
        type: emojiType!
      }
    };
  }
}
```

See it in action in this Stackblitz:

<app-stackblitz  
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Parsers-Emoji" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/parser_emoji.jpg" | relative_url }}"
></app-stackblitz>

## Example 3: Image parser

A really neat use-case for custom parsers is to take standard HTML elements and replace them with more useful Angular components.

For example, we could automatically add lightboxes to all images of an article marked with a `lightbox` class, so users can click on them to see a larger version. Assuming we have html like:

```html
<img class="lightbox" src="image.jpeg" src-large="image-large.jpeg">
```

A parser that automatically replaces those `<img>` elements with `ClickableImgComponent`s could then look as follows:

```ts
...
export class ClickableImgParser implements HookParser {

  findHookElements(contentElement: any): any[] {
    // Find all img-elements with the lightbox class
    return Array.from(contentElement.querySelectorAll('img.lightbox'));
  }

  loadComponent(): HookComponentData {
    return {
      component: ClickableImgComponent,   // Load our component
      hostElementTag: 'lightbox-img'      // As img-elements can't have content, replace them with '<lightbox-img>' elements
    };
  }

  getBindings(hookId: number, hookValue: HookValue): HookBindings {
    // Read the image urls from the element attributes and pass along as inputs
    const imgElement: HTMLImageElement = hookValue.elementSnapshot;
    return {
      inputs: {
        src: imgElement.getAttribute('src'),
        srcLarge: imgElement.getAttribute('src-large')
      }
    }
  }
}
```

Our `ClickableImgComponent` will then use `src` to render the image in the article itself and use `srcLarge` (if it exists) for the lightbox version.

See it in action in this Stackblitz:

<app-stackblitz 
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Parsers-Image" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/parser_image.jpg" | relative_url }}"
></app-stackblitz>

## Example 4: Link parser

Another cool idea for a custom parser is to enhance standard HTML links so that clicking on them uses the actual Angular router instead of reloading the whole browser tab, which is slow and costly.

We can simply write a custom <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> that looks for internal links in the content and automatically replaces them with a component that uses proper `[routerLink]`s.

Let's assume we have prepared a `DynamicLinkComponent` that renders a single Angular link based on the inputs `path`, `queryParams` and `fragment`. Here then, would be our custom <a href="https://github.com/Angular-Dynamic-Hooks/ngx-dynamic-hooks/blob/1a94c3517235a2b2d571379d1cfce88958cb3f66/projects/ngx-dynamic-hooks/src/lib/interfacesPublic.ts#L51" target="_blank">`HookParser`</a> for it:

```ts
import { Inject, Injectable } from '@angular/core';
import { DOCUMENT } from '@angular/common';
import { HookParser, HookValue, HookComponentData, HookBindings } from 'ngx-dynamic-hooks';
import { DynamicLinkComponent } from 'somewhere';

@Injectable({
  providedIn: 'root'
})
export class DynamicLinkParser implements HookParser {
  base;

  constructor(@Inject(DOCUMENT) private document: Document) {
    this.base = `${this.document.location.protocol}//${this.document.location.hostname}`;
  }

  public findHookElements(contentElement: HTMLElement): any[] {
    // First get all link elements
    return Array.from(contentElement.querySelectorAll('a[href]'))
    // Then filter them so that only those with own hostname are returned
    .filter(linkElement => {
      const url = new URL(linkElement.getAttribute('href')!, this.base);
      return url.hostname === this.document.location.hostname;
    });
  }

  public loadComponent(): HookComponentData {
    return { component: DynamicLinkComponent };
  }

  public getBindings(hookId: number, hookValue: HookValue): HookBindings {
    const url = new URL(hookValue.elementSnapshot.getAttribute('href')!, this.base);

    // Extract what we need from the URL object and pass it along to DynamicLinkComponent
    return {
      inputs: {
        path: url.pathname,
        queryParams: Object.fromEntries(url.searchParams.entries()),
        fragment: url.hash.replace('#', '')
      }
    };
  }
}
```

See it in action in this Stackblitz:

<app-stackblitz 
  url="https://stackblitz.com/github/Angular-Dynamic-Hooks/Example-v3-Parsers-Link" 
  fileQueryParam="file=src%2Fapp%2Fapp.component.ts"
  image="{{ "/assets/images/stackblitz/parser_link.jpg" | relative_url }}"
></app-stackblitz>