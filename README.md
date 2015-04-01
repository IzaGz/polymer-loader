# polymer-loader

Loads polymer-elements as first-class bundles. Allows you to separate components into a .js file, a .html file containg the `<polymer-element>` and a CSS file. Polymer-loader can be configured to use your choice of HTML/CSS preprocessor (Jade, EBS, Stylus, LESS, SASS, etc.).

## Usage

I suggest putting your components in their own directory, say `/components`

```
▾ components/
    ▾ foo/
        x-foo.html
        x-foo.js
        x-foo.styl
    ▸ some-element/
    ▾ charts/
      ▾ bar-chart/
          bar-chart.html
          bar-chart.js
          bar-chart.styl
```

Where each component consists of the 3 parts of a webcomponent. Your `Polymer('foo', {})` registration would go in the .js file, css would go in your css (or stylus/less/whathaveyou) file and your `<polymer-element name='...'>` goes in the .html file.

Now you will need to add a `preLoaders` entry to your `module` section in your `webpack.config.js`

```
module: {
  loaders: [
      ....
  ],
  preLoaders: [
    {test: /\/components\/.+\.js$/, loader: 'polymer-loader?templateExtension=html&styleExtension=styl'}
  ]
}
```

Change `templateExtension` & `styleExtension` to desired filetypes (don't forget to have the correct `loader` config for that extension!)


Finally, whenever/wherever you want to `require()` this module in your bundle or chunk, just call `require('../components/foo/foo.js')` (remember it's relative to the file you require from).

#### Goodies / Tips

**Extensionless, absolute require**

Here's a little function that generates aliases for the files in your `/components` directory, allowing you to require components like `require('components/foo')` and `require('components/foo/type/bar')` from any file in your application (without having to have every component's .js file named `index.js`).

```
        alias: (function() {

            var o = {}, root = __dirname + '/components/';

            function parse(nodes, parent) {

                if(parent && nodes.indexOf(path.basename(parent) + '.js') > -1)
                    o['components/'+parent] = __dirname + '/components/' + parent + '/' + path.basename(parent) + '.js';
                else
                    nodes.forEach(function(node) {
                        var path = root+(parent ? parent+'/' : '')+node;
                        if(fs.statSync(path).isDirectory())
                            parse(fs.readdirSync(path), (parent ? parent+'/' : '')+node);
                    });
            }

            parse(fs.readdirSync(root));

            return o;
        })()
```


**Stylus Imports**

`Stylus-loader` is nice enough to resolve `@imports` for us, so as long as you have your css/style directory listed in `modulesDirectories` you can add import common variable/helper stylus files into your component's `<style>` tag.

```
        modulesDirectories: ["node_modules", "bower_components", "css", "js"],
```

```
@import ~variables
@import ~themes
@import ~utils

foo
  $bar
  fooTheme()
```

***Registering Elements***

It may be advisable to register the element to register each element outside of `require()` statement. For example, if you a common base element that many other elements extend, it must be registered before the child elements and subsequent `require()` calls to ensure that it has been registered and that Polymer will not throw errors because it already has been.

my-component.html:
```
<polymer-element name='x-foo' extends='base-element'>
...
</polymer-element>
```

my-component.js:
```
var base = require('components/base-element');
var mixin = require('mixins/somestuff.js');

// ensureRegistered is a function you provide that calls Polymer(name, proto)
ensureRegistered('base-element', base);

var proto = {
   ... 
}

module.exports = Polymer.mixin(proto, mixin);
```

main.js:
```
function init() {
   // each component requires 'base-element' without duplication because it is a dependancy. 
   Polymer('my-component', require('components/my-component'));
   Polymer('my-component-other', require('components/my-component-other'));
}
```



More to come
