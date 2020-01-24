---
title: An organized approach to Webpack configuration
date: 2019-11-22 18:05:20
tags:
  - webpack
---

[Webpack](https://webpack.js.org) is an excellent tool in the web developers utility belt. It has become one of the expected technologies that any competent Web UI engineer is expected to be familar with, especially if they're using technologies like React. However, one of the biggest hurdles to using Webpack has been properly configuring it.

In this post, I am going to walk through my current webpack configuration for my personal [chatroom](https://github.com/justinzelinsky/chatroom) project. This project is what I work on in my spare time to hone my Web UI skills.

## How to think about Webpack Configuration

While Webpack technically works right out of the box and does not require any configuration file, almost all engineers will need to create one for any enterprise-level application. Your webpack configuration file is generally called `webpack.config.js` and is usually found in the root of your application.

It's important to note the filename -- `webpack.config.**js**`. It's a JavaScript file -- this means we're able to write JavaScript code when building our configuration object which Webpack uses. I believe we should heavily leverage this opportunity to make use of the fact we can script our configuration file and programatically make changes based on certain situations.

## `webpack.config.js`

### Imports

```javascript
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
const path = require('path');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const { DefinePlugin } = require('webpack');
```

At the top of your file, you'll likely be importing the various plugins or other libraries you'll be using when building your configuration. I recommend sorting your imports by their path, alphabetically. This is a general practice I recommend for two reasons:

1. You're able to quickly scan the code for what you're looking for if you have an understanding of the relative location of the line of code based on the path name.

2. You are less likely to run into merge conflicts with other team members if everyone is sorting things alphabetically -- lines of code tend to slip into their correct position by default when merging branches.

### Program variables

```javascript
const paths = {
  source: path.join(__dirname, 'src/ui'),
  dist: path.join(__dirname, 'dist')
};
const apiAddress = 'http://localhost:8082';
const devMode = process.env.NODE_ENV !== 'production';
```

One of the first things I do in my file is define the variables that will be used throughout the file. In this case, I have three:

- `paths`: A variable which contains all the paths needed for my configuration.In the above case, I have two paths -- `source`, located in `<rootDir>/src/ui` and `dist`, located in `<rootDir>/dist`. I can then access these paths via the `paths` object (e.g. `paths.source`)
- `apiAddress`: The address of the backend server that the UI will interact with
- `devMode`: A boolean which will tell whether or not webpack is running in production mode.

### Webpack variables

The following sections all represent different properties that exist on a webpack configuration object. In my configuration file, I write each property as an indepdent variable, in alphabetical order, and at the end of the file, I export all of the variables as one JavaScript object. It will become more obvious as to why I chose this approach as we go through each property.

#### entry

```javascript
const entry = path.join(paths.source, 'index.jsx');
```

This is the entrypoint of the webpack configuration. Here I am leveraging my `paths` variable and returning the location of the entrypoint of my application -- `<rootDir>/src/ui/index.jsx`.

#### devServer

```javascript
const devServer = {
  contentBase: paths.dist,
  compress: true,
  historyApiFallback: true,
  hot: true,
  open: 'Google Chrome',
  port: 9000,
  proxy: {
    '/api': apiAddress
  }
};
```

`devServer` represents the configuration object for `webpack-dev-server`. Explaining what each of these properties refers to is outside the scope of this article but you can read more at the [documentation](https://github.com/webpack/webpack-dev-server). Please note the references to `paths` and `apiAddress`.

#### devtool

```javascript
const devtool = devMode ? 'cheap-module-eval-source-map' : 'inline-source-map';
```

This is the dev tool to use for the webpack configuration. It leverages the `devMode` property to determine which built in devtool to use when webpack bundles your code.

#### mode

```javascript
const mode = devMode ? 'development' : 'production';
```

`mode` is the property to tell webpack whether or not it's in production mode (and should use various production-related optimizations).

#### rules

```javascript
const rules = [
  {
    test: /.(jsx?)$/,
    include: paths.source,
    exclude: /node_modules/,
    use: ['babel-loader', 'eslint-loader']
  },
  {
    test: /.scss$/,
    include: paths.source,
    use: [
      devMode ? 'style-loader' : MiniCssExtractPlugin.loader,
      {
        loader: 'css-loader',
        options: {
          importLoaders: 1,
          sourceMap: !devMode,
          modules: {
            mode: 'local',
            localIdentName: '[name]-[local]-[hash:base64:6]'
          }
        }
      },
      {
        loader: 'sass-loader',
        options: {
          prependData: '@import "src/ui/styles/vars";'
        }
      }
    ]
  },
  {
    test: /\.css$/,
    use: ['style-loader', 'css-loader']
  }
];
```

Here's where things get interesting -- this is our `rules` array. The `rules` array contains `rule` objects, each which represent how to treat different files that webpack comes across. So, for example, let's look at the first rule object defined:

```javascript
{
  test: /.(jsx?)$/,
  include: paths.source,
  exclude: /node_modules/,
  use: ['babel-loader', 'eslint-loader']
}
```

This rule is for files ending in either `.js` or `.jsx`. The `test` property is what determines which files this rule applies to. `include` and `exclude` refer to which directories that webpack should look for these files.

`use` is an array of loaders which will process these files. In the above example, we're using two loaders: `babel-loader` and `eslint-loader`. It's important to note that the order of the rules matter and files actually start at the end of the array and move to the beginning. So in the above example, a file (e.g. `utils.js`) will first be sent through the `eslint-loader` (which will run eslint against it), and then send it to `babel-loader` (which will transform it via Babel). Another way to think about this is to imagine the loaders as functions that wrap one another. e.g. `babelLoader(esLintLoader(file));`

Please note the use of the `paths` variable in `include`.

#### optimization

```javascript
const optimization = {
  minimize: true,
  minimizer: [
    new OptimizeCSSAssetsPlugin(),
    new TerserPlugin({ extractComments: false, parallel: true })
  ],
  splitChunks: {
    chunks: 'all'
  }
};
```

`optimization` is a configuration property which will be used when webpack is running in production mode. In the above configuration, we will be using two plugins (`OptimizeCSSAssetsPlugin` and `TerserPlugin`) and also splitting our code into chunks. Please refer to the webpack documentation for more details.

#### output

```javascript
const output = {
  filename: 'app.js',
  path: paths.dist
};
```

This property tells webpack where to output the results of the bundling process. Notice that we're using the `paths` property here.

#### plugins

```javascript
const plugins = [
  new CleanWebpackPlugin(),
  new HtmlWebpackPlugin({
    template: path.join(paths.source, 'index.html'),
    title: 'React/Redux Chatroom'
  }),
  new DefinePlugin({
    API_ADDRESS: JSON.stringify(apiAddress)
  })
];

if (!devMode) {
  plugins.push(
    new MiniCssExtractPlugin({
      filename: '[name].[hash].css',
      chunkFilename: '[id].[hash].css'
    })
  );
}

if (process.env.WEBPACK_ANALYZE) {
  plugins.push(new BundleAnalyzerPlugin());
}
```

Here's another interesting section. `plugins` refers to various webpack plugins which we want to use during our webpack build process. However, you'll notice that we conditionally add plugins. Thanks JavaScript!

In the above example, you see that we add three plugins by default:

- `CleanWebpackPlugin` - this cleans the destination folder (defined in `output`) before Webpack outputs anything new when it builds.
- `HtmlWebpackPlugin` - this plugin takes our `index.html` file and injects various properties into it. For example, we can have Webpack inject the title of the application. It will automatically inject any related css or javascript (or paths to the files) generated by webpack.
- `DefinePlugin` - this plugin allows us to define variables which webpack will scan the code for and replace instances of them with the value defined in the configuration for the plugin. In the above example, we have a variable called `API_ADDRESS`. This variable is used in the chatrooms clientside websocket connection and in order to make the UI agnostic to what API it's talking to, we leverage Webpack to replace the variable with the appropriate value. This could be toggled based on different environments.

You will then notice that we conditionally add two more plugins. First, if we're not in development mode, we add the plugin `MiniCssExtractPlugin` -- this will be used to extract our css into a separate css file and this file will be injected into the `<head>` of our HTML file outputed. The second plugin we add is the `BundleAnalyzerPlugin` -- this is added if the environment variable `WEBPACK_ANALYZE` has been defined. In this codebase, I have a npm script which defines this variable and then runs webpack. This plugin is used to analyze how your webpack is built and what libraries are using more space than others.

Thanks to the fact that this configuration file is pure JavaScript, we can condtionally add plugins by just writing standard JavaScript code. Pretty neat!

#### resolve

```javascript
const resolve = {
  extensions: ['.jsx', '.js'],
  modules: ['node_modules', paths.source],
  symlinks: false
};
```

`resolve` is an object which helps Webpack determine how to deal with module resolution. `extensions` is an array of file extensions that webpack can use when looking for files. For example, imagine I have the following import in my JavaScript code:

```javascript
import foo from './foo';
```

Webpack will search for `foo.jsx` and `foo.js` when trying to resolve this file path.

`modules` is an array of paths to aid with module resolution. In the above case, we're telling webpack that when it attempts to resolve a module path, you can search the `node_modules` or `paths.source`. The latter is very useful because you no longer need to write relative paths for local imports. For example, if you have all of your components living inside of a folder `src/ui/components`, you can import one component from another like this:

```javascript
import Button from 'components/Button';
```

and regardless of the location of the file with that import statement, Webpack will find the file for you.

`symlinks` being false is an optimization which you can read about [here](https://webpack.js.org/guides/build-performance/#resolving)

#### The final configuration object

```javascript
module.exports = {
  entry,
  devServer,
  devtool,
  mode,
  module: {
    rules
  },
  output,
  optimization,
  plugins,
  resolve
};
```

That's it! That's our webpack configuration object. We have defined all of our variables above and we can simply export them together in an object.

### Purpose

I hope you see the value I am proposing by breaking up the configuration into separate variables. These variables, being independent from one another, allow you to programtically alter or change them based on what environment you're in. You do not have to worry about overusing inline conditional statements or anything like that. The final object being exported appears really clean and also allows you to quickly glance and see what properties you're actually configuring -- this is much easier to deal with than an entire object being exported which could be over one hundred lines of code.
