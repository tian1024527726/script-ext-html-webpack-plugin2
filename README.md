## Installation
You can be running webpack v1.x or v2.x on node v4 or higher.

Install the plugin with npm:
```shell
$ npm install --save-dev script-ext-html-webpack-plugin
```

Note: you may see the following warning:
```shell
npm WARN html-webpack-plugin@XXX requires a peer of webpack@* but none was installed.
```
This is fine - for testing, we dynamically download multiple version of webpack (via the [dynavers](https://github.com/numical/dynavers) module).


## Basic Usage

### Use Case: Internalize all your JS
Add the plugin to your webpack config.

The order is important - the plugin must come **after** HtmlWebpackPlugin and ExtractTextWebpackPlugin:
```javascript
module: {
  loaders: [
    { test: /\.js$/, loader: 'babel-loader'}
  ]
}
plugins: [
  new ScriptExtHtmlWebpackPlugin()  << add the plugin
]
```
That's it.

Note that for this simple configuration, HtmlWebpackPlugin's [inject](https://github.com/jantimon/html-webpack-plugin#configuration) option must not be `false`.  However, this constraint does not apply if you specify the `position` - see 'Use Case: Specifying Position of Style Element' below


### Use Case: Internalize critical JS only
Add the plugin and use more than one loader for your JS:
```javascript
module: {
  loaders: [
    { test: /\.js$/, loader: 'babel-loader'}
  ]
}
plugins: [
  new HtmlWebpackPlugin({...}),
  new ScriptExtHtmlWebpackPlugin()
]
```

### Use Case: Internalize critical JS with all other JS in an external file
Use two instances of ExtractTextPlugin and tell ScriptExtWebpackPlugin which one to target by giving it the name of the output file:
```javascript
return {
  ...
  module: {
    loaders: [
      { test: /\.js$/, loader: 'babel-loader'}
    ]
  }
  plugins: [
    new HtmlWebpackPlugin({...}),
    new ScriptExtHtmlWebpackPlugin('internal.js') << tell the plugin which to target
  ]
}
```

### Use Case: Specifying Position of Script Element
In the above cases, the positioning of the `<script>` element is controlled by the [`inject` option](https://github.com/jantimon/html-webpack-plugin#configuration) specified by html-webpack-plugin.
For more control, you can use an extended, hash version of the configuration. This can have the following properties:
- `enabled`: [`true|false`] - for switching the plugin on and off (default: `true`);
- `file`: the js filename - previously, the single `String` argument (default: `undefined` - uses the first js file found in the compilation);
- `chunks`: which chunks the plugin scans for the js file - see the next Use Case: Multiple HTML files for usage (default: `undefined` - scans all chunks);
- `position`: [`head-top`|`head-bottom`|`body-top`|`body-bottom`|`plugin`] - all (hopefully) self-explanatory except `plugin`, which means defer to html-webpack-plugin's `inject` option (default: `plugin`);
- `minify`: see next section
- `jsRegExp`:  A regular expression that indicates the js filename (default: /\.js$/);

So to put the JS at the bottom of the `<head>` element:
```javascript
module: {
  loaders: [
    { test: /\.js$/, loader: 'babel-loader'}
  ]
}
plugins: [
  new HtmlWebpackPlugin({...}),
  new ScriptExtHtmlWebpackPlugin({
    position: 'head-bottom'
  })
]
```

### Use Case: Minification/Optimisation
The inlined JS can be minified/optimised using the extended, hash version of the configuration.  Use the `minify` property with one of the following values:
- `false`: the default, does not minify;
- `true`: minifies with default options;
- a hash of the minification options.
Minification is carried out by the [uglify-js](https://github.com/mishoo/UglifyJS2) optimizer 

Default minification:
```javascript
plugins: [
  ...
  new ScriptExtHtmlWebpackPlugin({
    minify: true
  })
]
```

Custom minification:
```javascript
plugins: [
  ...
  new ScriptExtHtmlWebpackPlugin({
    minify: {
      level: {
        1: {
          all: false,
          tidySelectors: true
        }
      }
    }
  })
]
```


### Use Case: Multiple HTML files
html-webpack-plugin can generate multiple html files if you [use multiple instances of the plugin](https://github.com/ampedandwired/html-webpack-plugin#generating-multiple-html-files).  If you want each html page to be based on different assets (e.g a set of pages) you do this by focussing each html-webpack-plugin instance on a particular entry point via its [`chunks` configuration option](https://github.com/ampedandwired/html-webpack-plugin#configuration).

script-ext-html-webpack-plugin supports this approach by offering the same `chunks` option.  As you also need an instance of extract-text-webpack-plugin, the configuration is quite unwieldy:
```javascript
...
const webpackConfig = {
  ...
  entry: {
    entry1: 'page-1-path/script.js',
    entry2: 'page-2-path/script.js'
  },
  output.filename = '[name].js',
  module.loaders: [
    {
      test: /\.js$/,
      loader: 'babel-loader',
      include: [
        'page-1-path'
      ]
    },
    {
      test: /\.js$/,
      loader: 'babel-loader',
      include: [
        'page-2-path'
      ]
    }
  ],
  plugins: [
    new HtmlWebpackPlugin({
      chunks: ['entry1'],
      filename: 'page1.html'
    }),
    new HtmlWebpackPlugin({
      chunks: ['entry2'],
      filename: 'page2.html'
    }),
    new ScriptExtHtmlWebpackPlugin({
      chunks: ['entry1']
    }),
    new ScriptExtHtmlWebpackPlugin({
      chunks: ['entry2']
    })
  ],
  ...
}
return webpackConfig;
```
