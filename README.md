# Blazor.JS.Component
An expansion to Blazor that adds the ability to painlessly attach a JS module counterpart to Razor components.

This library relies on [Blazor's JavaScript and C# interoperability](https://learn.microsoft.com/en-us/aspnet/core/blazor/javascript-interoperability/?view=aspnetcore-7.0) capabilities. I recommend familiarizing yourself with the JS interop API before using Blazor JS Components.

### Purpose
The library's main purpose is to provide a more developer-friendly way to interop between C# and JS on a per-component basis, making for a cleaner project and more readable code.

In some projecs you'd want to use JS libraries alongside your C# code, as most of them have no C# bindings. Blazor's current implementation of JS interop requires you to add a static JS file and load it by invoking "import" via JSRuntime.

This library does it automatically for you. All you have to do is add a `.razor.js` file to the same folder as your `.razor` component, and it will find the path, load the module, and call the "initializer" function with all the params you need for a (somewhat) clean and painless interop experience.


## Getting Started

[TBA - NuGet Package]

1. In your Razor component, add `@inherits JSComponent` at the top.
2. At the same folder as your componnet, add a JavaScript file with the same name as the component and a `.razor.js` extension. For example: `Counter.razor.js`. If you're using Visual Studio (not Code) you'll see it nested under your Razor component.
3. In your JavaScript file, add a function with the following signature: `export function ComponentName(rootElement, dotNetObjectRef)`.
4. In your Razor Component's HTML, wrap the content in an element and add `@ref="rootElement"` to it. The root element is necessary for selecting HTML elements relative to your component in the JS module.
5. Add any JS code you'd like to apply to your component to the function's body.

## Main Usage

### Handling a JavaScript event with a C# method

JavaScript:
```javascript
export function Counter(rootElement, dotNetObjRef) {
    root = rootElement;
    const button = rootElement.querySelector("#my-button");
    const hammer = new Hammer(button);
    hammer.on("pan", (ev) => {
        dotNetObjRef.invokeMethod("HandlePan", { deltaX: ev.deltaX, deltaY: ev.deltaY });
    });
}
```

C#:
```cs
public record DeltaParams(int deltaX, int deltaY);

[JSInvokable]
public void HandlePan(DeltaParams param)
{
    CurrentCount += param.deltaY;
    StateHasChanged();
}
```

In this example, I use the [hammer.js](https://hammerjs.github.io) library to add a `pan` event to my button, and handle the event in the C# counterpart of the component.

* Note that I extracted only the specific properties I needed, and added a corresponding record to C# to handle the values. This is necessary due to JS event objects having circular references and not being `JSON.stringify`-able. I recomment always creating a custom type, may it be a class or record, for handling event args.

* Also note how I used the `rootElement` along with `querySelector` to select a child relatively to the root. This way you can access the specific instance's children, rather than making global selectors.

### Getting a C# property value from JavaScript

Using the `GetProperty` method of `JSComponent`, you can get any property by name:

```javascript
const currentCount = dotNetObjRef.invokeMethod("GetProperty", "CurrentCount");
```
<i>DISCLAIMER: the `GetProperty` method uses reflection and it's not guaranteed to run smoothly when called in loops. It is recommended to implement a custom getter method for the purpose of multiple consecutive calls.</i>

Unfortunately due to type casting issues, if you want to set the property from JS, you'd have to implement a custom setter method:
```cs
[JSInvokable]
public void SetCurrentCount(int newCount) 
{
    CurrentCount = newCount;
    StateHasChanged();
}
```
And invoke it from the JS module:
```javascript
dotNetObjRef.invokeMethod("SetCurrentCount", 3);
```

