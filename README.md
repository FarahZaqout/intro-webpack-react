# Webpack and React
Webpack is a web bundler. Bundlers compile your code into one file and let the browser load it.

In order to efficiently use front end frameworks, we need a few things done before the browser can use it.

Browsers do not support the module `import` and `export` yet. To fix that we will be using `webpack`. Athough webpack provides more functionality and potetnial to how we can use web apps, this is the most important thing for now.

## Create React App:
running the command `npx create-react-app` sets up a react project for us. Behind the scene, this command uses webpack among other things to put everything together.

## Using Webpack:
We will be setting up our own react project without using `create-react-app`. So let's just jump into it.

### Create a directory.
We're going to create a directory and add webpack to it.

* `mkdir webpack-react`
* `cd webpack-react`
* `npm init -y`

for now, let's create a simple public folder and add an `index.html` and an `bundle.js` in it.

inside `index.html` add the following:
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Document</title>
    </head>
    <body>
        <script src="./index.js"></script>
    </body>
</html>
```

and inside`index.js` add the following code:
```javascript 
console.log('good news everyone we have webpack');
```

try opening the html file in the browser and look at the console output.

So far everything should be working as expected.

Now add the following to your `index.js`.

```javascript
import path from 'path'
```

refresh the page and check the console. You should see the following error:
`Uncaught SyntaxError: Cannot use import statement outside a module`

This is because browsers do not support `import` yet.

### Installing webpack:
in order to use webpack properly, we need to separate our `public` directory from our `src`.

* delete your bundle.js
* create a `src` folder and add `index.js` to it, with the same console.log and import statement from before.
* install webpack `npm i -D webpack webpack-cli`

now all we need is to tell webpack to create the `bundle` and use it in our index.html. To do that, we need a webpack config file: `touch webpack.config.js`

### Webpack config:
Inside the webpack config file, add the following code:
```javascript
    const path = require('path');

    module.exports = {
        entry: './src/index.js',
        output: {
            path: path.join(__dirname, 'public'),
            filename: 'bundle.js',
        },
    };

```

The config file exports a config object. Which tells webpack how to behave. It has an entry and an output keys. 

Entry is the file from which it will start reading our project's code, and output is where it will create the bundle. We will be adding more configs to it as we go, but for now this is all we need.
So let's check if this works.
* In index.html, change the script `src` attribute from `./index.js` to `./bundle.js`
* In package.json, add a start script: `"start": "webpack"`
* Run the start script and open the html file in liveserver. 
 
Things should be working without problems. Change the console string and check if the console changed in the browser. 

If you changed the console you have noticed it didn't change. This is because we have not created a new bundle. To do that, we have to run the start script again `npm start`.

### Webpack dev server

Running the start script every time we create something new is not a really good developer experience. We need webpack to automatically detect our changes and update the bundle. We can do that using webpack dev server. This will run a server for us, which will watch our files for any changes, re-bundle our code and serve the new bundle to us immediately.

* `npm i -D webpack-dev-server` and 
* update the start script to `"start": "webpack-dev-server --open"`

Running this script will not work yet. We still need to tell webpack to use the dev server.

* update the webpack.config.js
```javascript
const path = require('path');

module.exports = {
	entry: './src/index.js',
	output: {
		path: path.join(__dirname, 'public'),
		filename: 'bundle.js',
	},
	devServer: {
        // specifying the path from which the devserver will be serving the bundle.
		contentBase: path.join(__dirname, 'public'),
	},
};

```

Now any change we do in index.js will be reflected on our index.html. (We need to be on localhost:8080, we can also use the live server plugin in vscode).


## Adding React:
let's install react and react dom.
* `npm i react react-dom`
* inside index.html create the root div.
* in src/index.js add the following code:

```js
import React from 'react';
import ReactDOM from 'react-dom';

const element = React.createElement('p', null, 'Hello K3');

ReactDOM.render(element, document.getElementById('root'));

```

Now we will receive an error that the browser can't resolve the path to react. This is because import by package name (`import React from 'react'`) is not yet supported in the browser. To do this, we need a transpiler ([babel](https://babeljs.io/docs/)).

so let's install babel and use it. `npm i -D @babel/core babel-loader @babel/preset-env @babel/preset-react`

**so what are all these packages?**
* babel-core: Transforms your ES6 code into ES5
* babel-loader: Webpack helper to transform your JavaScript dependencies (for example, when you import your components into other components) with babel
* babel-preset-env: provide modern functionality on older browsers that do not natively support it
* babel-preset-react: Allows us to use jsx among other things.

now we need to do two things:
1- configure babel to use react.
2- tell webpack to use babel when building the bundle.

create a `.babelrc` file and add the following config in it:

```json
{
	"presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

now inside webpack add a `module` key to specify loader rules.
```javascript
const path = require('path');

module.exports = {
	entry: './src/index.js',
	output: {
		path: path.join(__dirname, 'public'),
		filename: 'bundle.js',
	},
	devServer: {
		contentBase: path.join(__dirname, 'public'),
	},
	module: { // new key
		rules: [
			{
				test: /\.(js|jsx)$/,
				exclude: /node_modules/,
				use: {
					loader: 'babel-loader',
				},
			},
		],
	},
};
```

Now that we have used the babel loader, read [this](https://webpack.js.org/guides/asset-management/#loading-css) and see if you can add css to webpack. Change the `React.createElement` syntax to jsx and add CSS to it.

### Adding a source map:
Source maps make debugging predictable. When an error is thrown, we usually have the bundle which is a mess. A source map will guarantee we can indentify where the error was thrown in the browser console.

There are different source maps, the simplest is the inline source map. To use it, add the following key to the webpack config object. `devtool: 'inline-source-map'`


### Using Different Configs:
The purpose of configuration for development and production is different. In development we want as many tools to help us make sure everything is good. However, in production, we only want the necessary packages. So we should have a different config file for each.

* create a `webpack.common.js` file. This will hold the configuration that we need in both dev and production environment. For example, loading CSS and babel should remain here.
* create a `webpack.dev.js`
* create a `webpack.prod.js`
* install a package called `webpack-merge` as a dev dependency. This package will help us merge the common and dev in development environment, and common with prod in production environment. `npm i -D webpack-merge`

Our common file will have the following set of configurations:

```javascript
const path = require('path');

module.exports = {
	entry: './src/index.js',
	output: {
		path: path.join(__dirname, 'public'),
		filename: 'bundle.js',
	},
	module: {
		rules: [
			{
				test: /\.js$/,
				exclude: /node_modules/,
				use: {
					loader: 'babel-loader',
				},
			},
		],
	},
};
```

our webpack.dev.js file would look like this:
```javascript
const path = require('path');
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
	mode: 'development',
	devtool: 'inline-source-map',
	devServer: {
		contentBase: path.join(__dirname, 'public'),
	},
});

```

Our weboack.prod.js would look like this:
```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
	mode: 'production',
});

```

Update your package.json scripts to look like this:
```json
"start": "webpack-dev-server --open --config webpack.dev.js",
"build": "webpack --config webpack.prod.js"
```

Try running the build and start script. They should be working fine.

The production is smaller because we no longer need the development crap. Now keep in mind this is the simplest webpack configuration with a tiny hello world project, so to appreciate what CRA does for us, here's a challenge.

---
### Challenge:

1- download custom fonts as project assets and try to get them to work. Make "Hello K3" appear in a font called league-spartan.
2- Add and load an image under "Hello K3"
3- Look up the following plugins:
    * clean webpack plugin.
    * html weboack plugin.
try to use them and make a judgment on whether they should be used in the common, prod or dev config file.


## Further Reading:
If you like webpack and want to experiment more with it, check out the [Official webpack guide](https://webpack.js.org/guides/getting-started/). It doesn't go into react per se, but it is the best available source to get a solid grip of webpack and how it works.
