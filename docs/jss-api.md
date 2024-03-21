# JavaScript API

## Access the global JSS instance

There is a global Jss instance which is the default export of the package. You can also create your own local Jss instance.

```javascript
import jss from 'jss'
```

## Quick setup with preset

```javascript
import preset from 'jss-preset-default'
import jss from 'jss'

jss.setup(preset())
```

## Create an own JSS instance

`jss.create([options])`

Use an own instance if the component you build should be reusable within a different project with a probably different JSS setup.

See `jss.setup()` below for `options` object description.

```javascript
import {create} from 'jss'
import camelCase from 'jss-plugin-camel-case'
import somePlugin from 'jss-some-plugin'

const jss = create()
jss.use(camelCase(), somePlugin())
jss.createStyleSheet(/* ... */)

export default jss
```

**Note**: `jss.create(options)` is the same as `jss.create().setup(options)`.

## Setup JSS instance

`jss.setup(options)`

Options:

- `createGenerateId` - a function which returns a function which generates unique class names.
- `plugins` - an array of functions, will be passed to `jss.use`.
- `Renderer` - if null, JSS will not render to DOM, or pass a custom Renderer.
- `insertionPoint` - string value of a DOM comment node which marks the start of sheets or a rendered DOM node. Sheets rendered by this Jss instance are inserted after this point sequentially.
- `id` - The options for the `createGenerateId`. This is an object which contains a single attribute called `minify` which should be a `boolean`.

**Note**: Each `jss.setup()` call will perform a shallow merge with the old options except for `plugins`. Passed `plugins` will get added to the existing plugins.

See [setup examples](./setup.md#specify-dom-insertion-point).

## Add a plugin

`jss.use(plugin)`

```javascript
import global from 'jss-plugin-global'
import jss from 'jss'

jss.use(global())
```

## Create Style Sheet

`jss.createStyleSheet([styles], [options])`

Classes get always generated by default.

Options:

- `media` - media query - attribute of style element.
- `meta` - meta-information about this style - attribute of style element, e.g., you could pass component name for easier debugging.
- `link` - link jss `Rule` instances with DOM `CSSRule` instances so that styles, can be modified dynamically, false by default because it has some performance cost.
- `element` - style element otherwise will create one by default.
- `index` - 0 by default - determines DOM rendering order, higher number = higher specificity (inserted after).
- `generateId` - a function that generates a unique class name.
- `classNamePrefix` - a string, which gets added at the beginning of the class name.

```javascript
import jss from 'jss'

const sheet = jss
  .createStyleSheet(
    {
      // "button" is a rule name; a class gets generated.
      button: {
        width: 100,
        height: 100
      }
    },
    {media: 'print'}
  )
  .attach()

console.log(sheet.classes.button) // button-d4f43g
```

```html
<style media="print">
  .button-0 {
    width: 100px;
    height: 100px;
  }
</style>
```

## Create a Style Sheet with global selectors

You need to have the [jss-plugin-global](https://github.com/cssinjs/jss/tree/master/packages/jss-plugin-global) plugin installed.

## Style Sheets registry

`SheetsRegistry`

When rendering on the server, you will need to get all rendered styles as a CSS string.
The `SheetsRegistry` class allows you to aggregate and stringify them. Read [more about SSR](ssr.md).

In case you are using the `SheetsRegistry` in the browser and you want to get Style Sheets which are `attached` or `detached` only, you can use the option `attached`.

If you want to remove whitespaces - use option `format`, which is `true` by default.

```javascript
import jss, {SheetsRegistry} from 'jss'

const sheets = new SheetsRegistry()
const sheet = jss.createStyleSheet()
sheets.add(sheet)
sheets.toString() // Returns all Style Sheets as a CSS string.
sheets.toString({attached: true}) // Returns all attached Style Sheets as a CSS string.
sheets.toString({attached: false}) // Returns all detached Style Sheets as a CSS string.
sheets.toString({format: false}) // Returns a CSS string without unnecessary whitespaces. Useful for SSR.
```

## Style Sheets Manager

`SheetsManager`

Counts how many elements use the same Style Sheet and automatically attach or detach it. It also acts similar to a WeakMap, because one can use an object as a key. React-JSS is using a `theme` object as a key to identify a sheet by a theme.

```javascript
import jss, {SheetsManager} from 'jss'

const manager = new SheetsManager()
console.log(manager.size) // 0
const sheet = jss.createStyleSheet()
const key = {}

manager.add(key, sheet) // index
console.log(manager.size) // 1
manager.get(key) // sheet

// Will attach the sheet and count refs.
manager.manage(key) // sheet
// Will detach the sheet if refs count is 0.
manager.unmanage(key)
```

## Remove a Style Sheet

`jss.removeStyleSheet(sheet)`

Detach the Style Sheet and remove it from the registry.

## Attach Style Sheet

`sheet.attach()`

Insert Style Sheet into the render tree. Call it to make your Style Sheet visible for the layout.

## Detach Style Sheet

`sheet.detach()`

Detaching unused Style Sheets will speed up every DOM node insertion and manipulation as the browser will have to do fewer lookups for CSS rules, which could potentially apply to the element.

## Attach Style Sheets in a specific order

Sheet 1 has a higher index (priority), and as such will come **after** sheet 2 in the resulting DOM.

```javascript
import jss from 'jss'

const sheet1 = jss.createStyleSheet({}, {index: 5, meta: 'sheet-1'}).attach()
const sheet2 = jss.createStyleSheet({}, {index: 1, meta: 'sheet-2'}).attach()
```

```html
<style data-meta="sheet-2"></style> <style data-meta="sheet-1"></style>
```

## Add a rule to an existing Style Sheet

`sheet.addRule(nameOrSelector, style, [options])`

This function will use [insertRule](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleSheet/insertRule) API if your Style Sheet is already attached to the DOM. In this case **you will not see this CSS Rule in the "Elements" view of the dev tools**. You can always verify correct rendering by using the selector on some DOM Node and watch styles applied as well as in the "Styles" inspector.

If you use `addRule()` before you call `attach()`, styles will be rendered in one batch using `textContent` API which will not have the above-described side effect.

## Replace a rule in an existing Style Sheet

`sheet.replaceRule(nameOrSelector, style, [options])`

Same as `sheet.addRule(...)` but replaces a rule with the same name if found.

### Options

- `index` - index where the rule should be added, by default, rules get pushed at the end.
- `className` - add a rule with a predefined class name.

### Add a rule dynamically

```javascript
import jss from 'jss'

const sheet = jss.createStyleSheet({})
const rule = sheet.addRule({
  padding: 20,
  background: 'blue'
})
document.body.innerHTML = `<button class="${rule.className}">Button</button>`
```

## Delete a rule from an existing Style Sheet

`sheet.deleteRule(name)`

To remove a rule from the DOM, Style Sheet option `link: true` should be used.
Returns `true` if the rule gets removed from the DOM.

## Get a rule

`sheet.getRule(name)`

Access a rule within sheet by a name.

```javascript
import jss from 'jss'

const sheet = jss.createStyleSheet({myButton: {}})
// Using name.
const rule = sheet.getRule('myButton')
```

## Add multiple rules

`sheet.addRules(styles)`

In case you want to add rules to the sheet separately or even at runtime.

```javascript
import jss from 'jss'

const sheet = jss.createStyleSheet({})
sheet.addRules({
  myButton: {
    float: 'left'
  },
  something: {
    display: 'none'
  }
})
```

## Update function values

`sheet.update(data)`

If you use [function values](./jss-syntax.md#function-values), you will want to update them with new data. This method will call all your function values, pass the `data` param and update the CSS Rule if needed.

```javascript
import jss from 'jss'

const styles = {
  container: {
    height: 200,
    width: (data) => data.width
  },
  button: {
    color: (data) => data.button.color,
    padding: (data) => data.button.padding
  }
}

const sheet = jss.createStyleSheet(styles, {link: true}).attach()

sheet.update({
  width: 100,
  button: {
    color: 'red',
    padding: 20
  }
})
```

## Create a rule without a Style Sheet

`jss.createRule([name], style, [options])`

Apply styles directly to the element but still be able to use JSS.

```javascript
import jss from 'jss'

const rule = jss.createRule({
  padding: 20,
  background: 'blue'
})
```

```js
import jss from 'jss'

const rule = jss.createRule('@media', {
  button: {
    color: 'red'
  }
})
```

## Apply a rule to an element inline

`rule.applyTo(element)`

This is equivalent to `element.style.background = 'blue'` except that you could use a rule from a sheet which is already defined. It uses `rule.toJSON()` internally, so same limitations are applied. [Example](http://cssinjs.github.io/examples/inline/index.html).

```javascript
import jss from 'jss'

const element = document.getElementById('element')
jss
  .createRule({
    background: 'blue'
  })
  .applyTo(element)
```

## Set or get a rule property dynamically

`rule.prop(name, [value])`

When the `link` option is true, after Style Sheet is attached, linker saves references to `CSSRule` instances so that you can set rule properties at any time. [Example](http://cssinjs.github.io/examples/dynamic-props/index.html).

```javascript
import jss from 'jss'

const sheet = jss.createStyleSheet(
  {
    a: {
      color: 'red'
    }
  },
  {link: true}
)

// Get the color.
console.log(sheet.getRule('a').prop('color')) // red

// Set the color.
sheet.getRule('a').prop('color', 'green')
```

## Convert a rule to JSON

`rule.toJSON()`

It returns a JSON representation of a rule. Supports only regular rules,
no nested, conditionals, keyframes or fallbacks.

The result of `toJSON` call can be used later to apply styles inline to the element.
It is used by `rule.applyTo()`.

## Convert to CSS

`sheet.toString()`

To get a pure CSS string from JSS, e.g. when preprocessing server side.

```javascript
import jss from 'jss'

const sheet = jss.createStyleSheet({
  button: {
    float: 'left'
  }
})

console.log(sheet.toString())
```

```css
.button-0 {
  float: left;
}
```

## Generate your class names

`createGenerateId`

Option `createGenerateId` allows you to specify a function which returns a class name generator function `generateId(rule, sheet)`. This pattern is used to allow JSS to reset the counter upon factory invocation when needed. For example, it is used in [React-JSS](https://github.com/cssinjs/jss/tree/master/packages/react-jss) to reset the counter on each request for server-side rendering.

By default class names generator uses a simple counter to ensure uniqueness of the class names. It consists of `classNamePrefix` Style Sheet option + rule name + counter. **Note**: in production (`NODE_ENV=production`) it uses just the `c` + rules counter.

```javascript
import jss from 'jss'

const createGenerateId = () => {
  let counter = 0

  return (rule, sheet) => `pizza--${rule.key}-${counter++}`
}

jss.setup({createGenerateId})

const sheet = jss.createStyleSheet({
  button: {
    float: 'left'
  }
})

console.log(sheet.toString())
```

```css
.pizza--button-1 {
  float: left;
}
```

## Minify selectors

When you want to minify your selectors in production for example, you can configure this in your `jss.setup` call.

> Note: This is disabled by default.

```js
import jss from 'jss'
// Pass the id option to jss.setup and set minify to true.
jss.setup({id: {minify: true}})
```

## Extract dynamic styles

`getDynamicStyles(styles)`

Extracts a styles object with only props that contain function values. Useful when you want to share a static part between different elements and render only the dynamic styles separate for each element.

```javascript
import {getDynamicStyles} from 'jss'

const dynamicStyles = getDynamicStyles({
  button: {
    fontSize: 12,
    color: (data) => data.color
  }
})

console.log(dynamicStyles)
// {
//   button: {
//     color: data => data.color
//   }
// }
```

## Plugins

See [plugins](plugins.md) documentation.