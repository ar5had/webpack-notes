# webpack-notes
simple webpack notes.

## TOC

- [Need of webpack](#need-of-webpack)
- [Basics](#basics)

## Need of Webpack

Webpack is a build tool that puts all of your assets, including Javascript, images, fonts, and CSS, in a dependency graph. In short it is an optimized advanced task runner that is used to process input file into output files. It supports all the three module system i.e., CommonJS, AMD, ES6 modules.

The process involves below steps:

*Transpilation + Concatenation + Minification + Combine CSS into JS (Extra-feature that other task runners dont have)*

![Process Diagram](./images/webpack-process.png)

**Note:** Webpack use NPM not Bower.

## Basics

Let's suppose we have a Project structure like -
```
// Project Structure
webpack project
--> app.js
--> index.html  
```

### To build a file into output file -

`webpack <input_file_path> <output_file_path>`


It is very tedious task to pass parameters everytime in command line, so another way to do such task is to define a *config* file. The name of config file will be `webpack.config.js` which will export a CommonJS module encapsulating all our settings for webpack.

``` js
module.exports =  {
  entry: "<entry_file_path>",
  output: {
    filename: "<output_file_path>"
  }
};

```
Now we just have to write `webpack` in console to build output file. It is kind of convention to name output file as `bundle.js`.

After running `webpack` in CLI -

```
// Project Structure
webpack project
// this is input file
--> app.js
--> index.html  
--> bundle.js
```

### To rerun build whenever your input file is changed -

In command line, type `webpack --watch`

or

change your config file to

``` js
module.exports =  {
  entry: "<entry_file_path>",
  output: {
    filename: "<output_file_path>"
  },
  watch: true
};

```
This will automatically make a new build whenever your input file is changed.

### To start server -

Webpack has a dev server which can be installed by below command -

`npm install webpack-dev-server -g`

After install, you can run your webpack-dev-server by below command -

`webpack-dev-server`

Now point your browser to `localhost:8080/webpack-dev-server/`. you will see you app running. To get the hot reload working, webpack is running your app in iframe. You will also be seeing a banner with `App ready`.

To get rid of banner and iframe, use below command -

`webpack-dev-server --inline`


### Building multiple files

```
// Project Structure
webpack project
--> app.js
--> index.html
--> login.js
--> global.js  
```

Lets suppose `index.html` has a script tag which corresponds to app.js, `app.js` requires `login.js` and `global.js` have some global variable and script that we need gloabally and we want to add in global scope.

To do so, change `webpack.config.js` file to -
``` js
module.exports =  {
  entry: ['global.js', 'app.js'],
  output: {
    filename: 'bundle.js'
  },
  watch: true
};

```

Project structure after running `webpack-dev-server` -
```
// Project Structure
webpack project
--> app.js
--> index.html
--> login.js
--> global.js  
--> bundle.js
```

### Loaders

Webpack doesnt know much things to do so we use loaders to do some other fancy stuff using `loaders`.

Loaders can be used to convert es6 to es5, jsx to js etc. To use them we will have to install individual loader.

**NOTE:** loaders must be added with `-dev` tag while installing as they are dev dependencies.

Lets add some loader modules -

After installing some loader, package json will look something simlir to below file -

```
{
  "name": "webpack-project",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {},
  "devDependencies": {
    "babel-core": "^6.21.0",
    "babel-loader": "^6.2.10",
    "babel-preset-es2015": "^6.18.0",
    "jshint": "^2.9.4",
    "jshint-loader": "^0.8.3",
    "node-libs-browser": "^2.0.0"
  }
}
```

**node-libs-browser** - node-libs-browser is a peer dependency of Webpack. As stated in the package page it provides certain Node libraries for browser usage. Obviously modules such as fs won't be available there but you can still use quite a few.

**js-hint** - basically it gives errors and warning depending upon code error and format.


Lets suppose we have added some es6 code in `login.js` adn renamed it to `login.es6` then to use loaders we can modify our `webpack.config.js` like below -

```
module.exports =  {
  entry: ['global.js', 'app.js'],
  output: {
    filename: 'bundle.js'
  },
  // for a loaders we add a module object
  module: {
    // array that holds loader's setting
    loaders: [
    // there are three key for each loader
    // test - tells which file to target (regex)
    // exclude - tells which file to ignore (regex)
    // loader - tells which loader to use
      {
        test: /\.es6$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  },
  // this object tells which type of files webpack can process without
  // specifically giving them file extension

  // webpack by default add .js extension but it can overridden, as we are
  // doing below
  resolve: {
    // notice we changed the login.js to login.es6
    // and we are requiring login file in app.js
    // so by adding below line webpack will first search for file
    // with name login then login.js and if that's not found too , it will
    // search for login.es6
    extensions: ['', '.js', '.es6']
  },

  watch: true
};

```

Now going back to console, we can type `webpack` and we will get bundle.js as output with all es6 code converted to es5. Similary react code can be transpiled too.

**NOTE:** Be careful about the spelling of keys in webpack.config.js


### Preloaders

Preloaders run before loaders. They are used to check linting errors etc. We are going to use jshint. First of all we create a `.jshintrc` file with below content -

```
// .jshintrc content
{}
```

In `webpack.config.js`

```
module.exports =  {
  entry: ['global.js', 'app.js'],
  output: {
    filename: 'bundle.js'
  },
  // for a loaders we add a module object
  module: {
    // array that holds loader's setting
    preLoaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'jshint-loader'
      }
    ],
    // array that holds loader's setting
    loaders: [
    // there are three key for each loader
    // test - tells which file to target (regex)
    // exclude - tells which file to ignore (regex)
    // loader - tells which loader to use
      {
        test: /\.es6$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  },
  // this object tells which type of files webpack can process without
  // specifically giving them file extension

  // webpack by default add .js extension but it can overridden, as we are
  // doing below
  resolve: {
    // notice we changed the login.js to login.es6
    // and we are requiring login file in app.js
    // so by adding below line webpack will first search for file
    // with name login then login.js and if that's not found too , it will
    // search for login.es6
    extensions: ['', '.js', '.es6']
  },

  watch: true
};

```

### Creating a Start script

Gulp, grunt have functionality to add custom commands to start script but webpack doesnt have so. That doesnt mean webpack is low on this side because npm has this functionality built in.

To run our server by custom npm command, goto package.json and do the following changes -

```
{
  "name": "webpack-project",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  // scripts allows to add custom commands
  "scripts": {
    "start": "webpack-dev-server"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {},
  "devDependencies": {
    "babel-core": "^6.21.0",
    "babel-loader": "^6.2.10",
    "babel-preset-es2015": "^6.18.0",
    "jshint": "^2.9.4",
    "jshint-loader": "^0.8.3",
    "node-libs-browser": "^2.0.0"
  }
}

```


now run your server just by typing `npm start` in command line.


### Production vs Develoment Builds
In development builds we dont want out code to be minimized for debugging reasons, but in production env we need to minimize the code to improve performance.

It is very simple to do by command-line -

`webpack -p`

-p tag will minimize the code We dont have to do these things by ourselves. Travis CI or other CI software will do that for you.

#### To create different config file to strip out some code
Now we are goin to install a loader named `strip-loader`

After installing it as dev dependency, we are going to make a production webpack config file named `webpack.production.config.js`.

Content -

```
// webpack.production.config.js
var WebpackStrip = require('strip-loader');
var devConfig = require('./webpack.config.js');
var stripLoader = {
  test:[/\.js$/, /\.es6$/],
  exclude: /node_modules/,
  // add statements here that are to be stripped out of code
  loader: WebpackStrip.loader('console.log')
}

devConfig.module.loaders.push(stripLoader);

module.exports = devConfig;

```
To run in production env, we can type a command like

`webpack --config=webpack.production.config.js -p`

--config value will tell which webpack config file to execute.

Note here we cant use `webpack-dev-server` as it will run and rebuild in dev environment so we are using `http-server` to create web server in cwd by typing command -

`http-server`

now going back to browser on right port, we will see tht console.log statement will not be executed.

We need to add a `.babelrc` file so that babel can work nicely with `UglifyJs`. Give it the following content -

`{ "presets": ["es2015"] }`

**A quick note about rc file** - rc dotfiles are configuration files that can vary in their use, formatting, and overall meaning. You can create .[whatever name you like]rc files to inform whatever package you happen to be creating (provided another package isn't looking for the same one). Usually, they're useful for some sort of tool that acts on your source code and needs some tuning specific to your project. My understanding is that there were similar files that played an important role in UNIX systems in years past and the idea has stuck.
