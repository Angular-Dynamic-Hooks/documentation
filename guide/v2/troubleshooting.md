---
canonical_url: '/guide/troubleshooting'
title: Troubleshooting
description: This page goes over the most common problems and errors you might encounter when using the Angular Dynamic Hooks library and suggests solutions.
---

# Troubleshooting

### **Some of my elements/attributes are not rendering!**

This might be due to sanitization. This library uses Angular's native DomSanitizer to remove potentially malicious code like `<script>`-tags from the content. To ensure maximum security, the sanitizer is fairly aggressive, however, and will also remove seemingly harmless elements, like `<input>` or attributes like `id`.

You can turn off sanitization at any time through the [OutletOptions]({{ "guide/v2/configuration#outletoptions" | relative_url }}). Note that you will then have to ensure that the content is safe to render by yourself!

### **Error: `ngx-dynamic-hooks` is not a known element**

Some editors like VS Code don't always immediately catch on to the newly available components when a module has been imported. Try restarting the editor to see if that helps (it should compile fine, though). If not, check that you have correctly imported the `DynamicHooksModule` into you main module as shown in the [Quick start]({{ "guide/v2/quickstart" | relative_url }})-section to make everything available.

### **Error: Data type for following input was not recognized and could not be parsed**

You most likely have a typo in the input. If its a string, remember to put quotation marks around it ('', "" or ``). If that isn't it, it may help to copy the input into an IDE that is set to JS/TS syntax and have it highlight potential typos for you.

### **In my output function, `this` does not point to the parent object of the function**

See the [Outputs-section]({{ "guide/v2/dynamic-component-features#outputs" | relative_url }}) for a solution to this problem.

### **globalParsersBlacklist/whitelist for the `OutletComponent` doesn't work**

Make sure you have explicitly given the parsers a name (see the [Parsers]({{ "guide/v2/parsers" | relative_url }})-section on how to do so) that correlates with the black/whitelisted name.

### **I'm writing a custom parser. When implementing `loadComponent()`, why are there placeholder-elements in the passed `childNodes`?**

At this point in the workflow, the original hooks have already been replaced with the placeholder-elements you see in the `childNodes`. These placeholders are later replaced again with the actual Angular components. 

Note that if you replace the inner content of the hook and modify or remove these placeholders, the corresponding component may not load correctly!

### **I've written a custom parser. `ngOnChanges()` keeps triggering**

It is important to remember that `getBindings()` on hook parsers is called anytime the current values of the bindings are requested. By default, that is on component creation and on every change detection run afterwards. If this function parses the bindings from scratch and returns new references for them each time it is called, the bindings are considered to have changed and `ngOnChanges()` in the dynamic components will be triggered (or in the case of an output binding, it will be resubscribed). 

You can avoid that by introducing a persistent state in your parsers and by remembering and reusing the previous references if they haven't changed. 

If you need a way to tell if the bindings are deeply identical by value for this, you can import the <a href="https://github.com/angular-dynamic-hooks/ngx-dynamic-hooks/blob/9b31ba5872a057c33a5464f638ac234fd6144963/projects/ngx-dynamic-hooks/src/lib/utils/deepComparer.ts" target="_blank">`Deep Comparer`</a> service from this library and use the `isEqual()` method (or alternatively use Underscore's <a href="https://underscorejs.org/#isEqual" target="_blank">isEqual()</a> or Lodash's <a href="https://lodash.com/docs/#isEqual" target="_blank">isEqual()</a>.

If you don't want to bother with any of that, you can also simply set the `compareInputsByValue`/`compareOutputsByValue`-options in <a href="https://github.com/angular-dynamic-hooks/ngx-dynamic-hooks/blob/9b31ba5872a057c33a5464f638ac234fd6144963/projects/ngx-dynamic-hooks/src/lib/components/outlet/options/options.ts" target="_blank">`OutletOptions`</a> to true (see [OutletOptions]({{ "guide/v2/configuration#outletoptions" | relative_url }})), which does this automatically, though it will then apply to all active parsers.

### **Error: TypeError: Object(…) is not a function**

You might be using Rxjs-version that is older than 6, which was introduced with Angular 6. If you are using Angular 5, either upgrade to 6 or try using <a href="https://www.npmjs.com/package/rxjs-compat" target="_blank">Rxjs compat</a> to fix this issue.