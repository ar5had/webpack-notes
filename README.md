# webpack-notes
Simple webpack notes. Reading these notes along with [Webpack Fundamental Course](https://app.pluralsight.com/library/courses/webpack-fundamentals/table-of-contents) by Joe Eames will benifit you the most. Feel free to fork it add make a pr to add your notes.

## TOC

- [Need of webpack](#need-of-webpack)
- [Basics](#basics)
- [Advanced](#advanced)
- [Adding CSS](#adding-css)
- [Adding Images & Fonts](#adding-images-&-fonts)
- [Webpack Tools](#webpack-tools)
- [Webpack and Front End Frameworks](#webpack-and-front-end-frameworks)

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

``` js
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

``` js
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

``` js
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

``` js
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
 js
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

## Advanced

### Organizing Files and Folders
We will not want to add bundle.js file in our source control. To do so we must add bundle.js file to a folder that is ignored by our source control but can be accessible through a pseudo relative url. We'll be running build command on server using travis ci or other ci, so it will be accessible on server. We just dont want to add the user's bundle.js file to the file that is used on server.

To do so we have to do some changes in our code.

Our new project structure looks like -
```
webpack-project

--> js
----> app.js
----> login.es6
----> utils.js

--> public
----> index.html

--> webpack.config.js
--> package.json
```

Our index.html files now looks like below:
```
<html>
  <body>
    <script src="/public/assets/js/bundle.js"></script>
  <body>
</html>
```

Practically there will be no file at `/public/assets/js/bundle.js`, what webpack-dev-server will do here is that it will serve file in `/build/js/bundle.js` to every request for the url `/public/assets/js/bundle.js`. If we dont use `webpack-dev-server` for build then we can't be able to serve file from pseudo path.

So we have to do below changes in `webpack.config.json` -

``` js
var path = require('path');


module.exports = {
  // Context key set relative path for entry key
  // earlier we didnt have directories so there was
  // no need of this key
  context: path.resolve('js'),
	entry: ['./utils.js','./app.js'],
	output: {
    // path key tells webpack where to put build file
    path: path.resolve('build/js/'),
    // publicPath key tells webpack the url for which
    // bundle file will be served
    publicPath: '/public/assets/js/',
		filename: 'bundle.js'
	},
  // as we have added folder now, so for webpack to serve
  // html, we need to tell which folder to look in
  // this is done by adding new key devServer
  devServer: {
    // it tells the path of static content
    contentBase: 'public'
  },

	module: {
		preLoaders: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: 'jshint-loader'
      }
    ],
		loaders: [
			{
				test: /\.es6$/,
				exclude: /node_modules/,
				loader: 'babel-loader'
			}
		]
	},
	resolve: {
		extensions: ['', '.js', '.es6']
	},
	watch: true

}

```

One interesting thing to mention here is , if we run `webpack-dev-server` and goto `localhost:8080/webpack-dev-server/index.html` build file will be created virtually and will not take space on disk.

If we run `webpack` command then disk file for bundle.js will be created but not when using `webpack-dev-server` command.


### Working with ES6 modules

Let's say we have changed our app.js file to app.es6 file and did some changes in login.es6 too.

Changes in these files -

app.es6
``` js
//require('./login');
import {login} from './login';
login("from app.js");
document.write("Hello, from the other side");
console.log('app loaded');

```

login.es6
``` js
let login = (a) => {
  console.log('ES6 code executed!', a);
  document.write("from login");
};

export {login};

```


Now to support es6 imports we have to do some small changes in `webpack.config.js` file.

Changes -

``` js
var path = require('path');


module.exports = {


  context: path.resolve('js'),
	// we have removed the extention from here
	// as we have already set resolve for extention below
	entry: ['./utils','./app'],


  output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
		filename: 'bundle.js'
	},

  devServer: {
    contentBase: 'public'
  },

	module: {
		loaders: [
			{
				test: /\.es6$/,
				exclude: /node_modules/,
				loader: 'babel-loader'
			}
		]
	},
	resolve: {
		extensions: ['', '.js', '.es6']
	}

}


```

Now running webpack-dev-server we will get our app going using es6 import.

### Adding Source Maps
Source maps are built into webpack, it can be added by adding `-d` flag in webpack and webpack-dev-server command.

Just add `debugger;` to part of code which you want to set up as breakpoint.


### Creating Multiple Bundles

We have changed our project structure, now our project structure looks like -

```
webpack-project

--> js
----> a.js
----> b.es6
----> c.js

--> public
----> a.html
----> b.html
----> c.html

--> webpack.config.js
--> package.json
```

Each of html file has two script tags. For ex - a.html has script tags to `/public/assets/js/a.js` and `/public/assets/js/shared.js`. Similar is the case with `b.js` and `c.js`.

Earlier we used to have on single file that has all the code that even wasn't needed to be there. Every thing loaded up before their use. Using this seperate bundle way, we can make lazy loading i.e., using code only at the time of need, this improves our app's performance.

Shared file will have shared code and other files will have their respective code.

But how do webpack know which file is for which html page??

To make webpack understand we have to do the following changes in `webpack.config.js` -

``` js
var path = require('path');
// adding webpack module so that we can make shared code in a file
var webpack = require('webpack');
// this code adds a plugin
var commonsPlugin = new webpack.optimize.CommonsChunkPlugin('shared.js');


module.exports = {
  context: path.resolve('js'),
  // replacing entry value with object
  entry: {
    a: './a.js',
    b: './b.js',
    c: './c.js'
  },
	output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
    // it will make the name of the output
    // file match the keys in entry
    filename: '[name].js'
	},

  plugins: [commonsPlugin],

  devServer: {
    // it tells the path of static content
    contentBase: 'public'
  },

	module: {
		loaders: [
			{
				test: /\.es6$/,
				exclude: /node_modules/,
				loader: 'babel-loader'
			}
		]
	},
	resolve: {
		extensions: ['', '.js', '.es6']
	}

}
```

We will creates seperate bundle for each file and shared code will be present in one file. Shared code have common modules that are imported in our code and other common stuff.

## Adding CSS

We are going to load css/less/sass using webpack. In order to do that we will need some loaders.

### Adding plain css

Our project structure at this time looks like -
```
webpack-project

--> css
----> index.css

--> js
----> app.js

--> public
----> index.html

--> webpack.config.js
--> package.json
```


Dont add style tags in html but add require them like js in `app.js`.

For plain css we need `style-loader` and `css-loader`.

To install them -

`npm install --save-dev style-loader css-loader`


Our index.html looks like -

``` js
<body>
  <h3>
    This is h3 heading;
  </h3>
</body>
// it is very necessary to add this script
// other wise styles wont load
<script src="/public/assets/js/bundle.js">

</script>

```

Add following changes to your `webpack.config.js` -

``` js
var path = require('path');


module.exports = {
  context: path.resolve('js'),
  entry: ["./app.js"],
	output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
    filename: 'bundle.js'
	},
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
  			test: /\.css$/,
        exclude: /node_modules/,
        // this means as soon as webpack encounters any css
        // file it will first goto css-loader and then style-loader
        loader: "style-loader!css-loader"
      }
    ]
	}

}

```

Now run `webpack-dev-server` command and goto brwoser , you will see css will be added into inline style tags.

### Adding SASS
Our project structure at this point of time looks like -
```
webpack-project

--> css
----> index.sass

--> js
----> app.js

--> public
----> index.html

--> webpack.config.js
--> package.json
```
We just have to install `sass-loader` and `node-sass` module and change `webpack.config.js` and `app.js` file.

Changes in `webpack.config.js` -

``` js
var path = require('path');


module.exports = {
  context: path.resolve('js'),
  entry: ["./app.js"],
	output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
    filename: 'bundle.js'
	},
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
  			test: /\.sass$/,
        exclude: /node_modules/,
        // this means as soon as webpack encounters any css
        // file it will first goto sass-loader then css-loader and then style-loader
        loader: "style-loader!css-loader!sass-loader"
      }
    ]
	}

}


```

Our app.js file now looks like -

``` js
require('../css/index.sass');
console.log('App loaded!');
```


Simlarly `less` can be used with webpack too.


### Creating Seperate CSS bundle
To do this we have to install `extract-text-webpack-plugin`.

Then we have to do some changes to `webpack.config.js`.

Changes -
``` js
var path = require('path');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  context: path.resolve('js'),
  entry: ["./app.js"],
	output: {
    path: path.resolve('build/'),
    publicPath: '/public/assets/',
    filename: 'bundle.js'
	},
  plugins: [
    // takes name of css file
    new ExtractTextPlugin("styles.css")
  ],
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
  			test: /\.sass$/,
        exclude: /node_modules/,
        // second argument must have booth
        // sass loader and css loader
        loader: ExtractTextPlugin.extract( "style-loader", "css-loader!sass-loader")
      }
    ]
	}

}

```

Now we have to do some changes in `index.html`.

Changes -
```
<link rel="stylesheet" href="/public/assets/styles.css">
<body>
  <h3>
    This is h3 heading;
  </h3>
</body>
<script src="/public/assets/bundle.js">

</script>

```


### Auto Prefixer
To use auto prefixer for different vendors, we have to install a module named `autoprefixer-loader` and then we have to do some changes to our `webpack.config.js` .

Changes in webpack.config.js
``` js
...

module: {
  loaders: [
    {
      test: /\.sass$/,
      exclude: /node_modules/,
      // simply add !autoprefixer-loader to
      // second argument
      loader: ExtractTextPlugin.extract( "style-loader", "css-loader!autoprefixer-loader!sass-loader")
    }
  ]
}
...

```

## Adding images & fonts

We have to install `url-loader` module, and add that loader in `webpack.config.js` and do some changes in `index.html` and `app.js`.

index.html
```
<link rel="stylesheet" href="/public/assets/styles.css">
<body>
  <h3>
    This is h3 heading;
  </h3>

  // added this element which will have image
  <span id="img">
  </span>

  <span>above image is loaded by webpack</span>
</body>
<script src="/public/assets/bundle.js">

</script>

```


webpack.config.js
``` js
var path = require('path');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  context: path.resolve('js'),
  entry: ["./app.js"],
	output: {
    path: path.resolve('build/'),
    publicPath: '/public/assets/',
    filename: 'bundle.js'
	},
  plugins: [
    new ExtractTextPlugin("styles.css")
  ],
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
  			test: /\.sass$/,
        exclude: /node_modules/,
        loader: ExtractTextPlugin.extract( "style-loader", "css-loader!autoprefixer-loader!sass-loader")
      },
      // added this new loader for images and fonts
      {
        test: /\.(png|jpg)$/,
        exclude: /node_modules/,
        // we pass parameter by give question
        // mark and then parameter and its
        // value. This limit parameter tells
        // webpack that below this limit make
        // images x64 encoded data other wise make
        // seperate request (dont give limit greater than 10kb)
        loader: 'url-loader?limit=100000'
      }
    ]
	}

}

```


Similarly font can be added just by adding `ttf` or `eot` etc in `test` of the `url-loader` loader.

Only change the css file -

```
// app.css
// this font will be loaded by webpack..
// if size is below the limit, then it will loaded
// inline in base 64 encoding
@font-face {
    font-family: myFirstFont;
    src: url(sansation_light.ttf);
}

body {
    font-family: myFirstFont;
}
```

## Webpack Tools

We are now going to rock our webpack with express server. To do so we need express, ejs and morgan as dependency and webpack-dev-middleware as dev dependency.

So after installing all those, we have to do some changes.

Our project structure is -
```
webpack-project

--> public
----> css
------> styles.css

----> js
------> app.js

--> views
----> index.ejs

--> routes
----> index.js

--> server.js
--> webpack.config.js
--> package.json

```

server.js code -
``` js
var express = require('express');
var path = require('path');
var logger = require('morgan');
var routes = require('./routes/index');

var app = express();

app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');

app.use(logger('dev'));

app.use('/', routes);

// Only load this middle in dev module
if (app.get('env') === 'development') {
    var webpackMiddleware = require("webpack-dev-middleware");
    var webpack = require('webpack');

    var config = require('./webpack.config');

    app.use(webpackMiddleware(webpack(config), {
      // overriding public path for index.ejs
      publicPath: "/build",
      headers: { "X-Custom-Webpack-Header": "yes"},
      stats: {
        color: true
      }
    }));
}

app.use((req, res, next)=>{
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

var server = app.listen(8000, () => {console.log('listening on 8000')});

```

index.ejs code -

```
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
     <!-- as public path is overridden in server.js -->
    <link rel="stylesheet" href="/build/styles.css">
    <title><%= title %></title>
  </head>
  <body>
    <h1><%= title %></h1>
    <!--  as public path is overridden in server.js -->
    <script src="/build/bundle.js"></script>
  </body>
</html>

```

webpack.config.js -
``` js
var path = require('path');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  context: path.resolve('public/js'),
  entry: ["./app.js"],
	output: {
    path: path.resolve('build/'),
    publicPath: '/public/assets/',
    filename: 'bundle.js'
	},
  plugins: [
    // takes name of css file
    new ExtractTextPlugin("styles.css")
  ],
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
  			test: /\.css$/,
        exclude: /node_modules/,
        loader: ExtractTextPlugin.extract( "style-loader", "css-loader")
      }
    ]
	}

}

```

Now to start server , just type `node server` and an express server will start.


### Creating a Custom Loader
We are creating a loader that strips comments from json. Comments in json are evil as they break `require`. We have to load some dev-dependencies `load-json` and `strip-json-comments` for this loader.

Our project structure looks like -
```
webpack-project

--> public
----> index.html

----> js
------> app.js
------> cf.json

--> loader
----> strip.js


--> webpack.config.js
--> package.json

```
index.html code

```
<body>
  <script src="/public/assets/js/bundle.js">

  </script>
</body>

```

app.js code
``` js
console.log('app loaded');
var c = require("./cf.json");
console.log(c);
require('../css/styles.css');

```

cf.json
``` js
// here is awesome comment

{
  "name": "Arshad"
}

```

stip.js code
```
console.log('app loaded');
var c = require("./cf.json");
console.log(c);
require('../css/styles.css');

```

webpack.config.js code
``` js
var path = require('path');
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
  context: path.resolve('js'),
  entry: "./app.js",
	output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
    filename: 'bundle.js'
	},
  plugins: [
    new ExtractTextPlugin("styles.css")
  ],
  devServer: {
    contentBase: 'public'
  },


	module: {


  	loaders: [
      // this loader is our custom loader
      {
        test: /\.json$/,
        exclude: /node_modules/,
        // puts path of custom loader that we created!
        loader: "json-loader!" + path.resolve('loader/strip')
      },


      {
  			test: /\.css$/,
        exclude: /node_modules/,
        loader: ExtractTextPlugin.extract( "style-loader", "css-loader")
      }
    ]
	}

}

```

### Using plugins
We ae going to install a dev dependency `timestamp-webpack-plugin` and dependency `jquery` which we will add globally throughout all the modules.

app.js code
``` js
console.log('app loaded');
var c = require("./cf.json");
console.log(c);
// $ must be present globally
$("#main").text("modified by jquery");
require('../css/styles.css');
```

webpack.config.js code
``` js
var path = require('path');
var ExtractTextPlugin = require('extract-text-webpack-plugin');
var webpack = require('webpack');
var TimestampPlugin = require('timestamp-webpack-plugin');

module.exports = {
  context: path.resolve('js'),
  entry: "./app.js",
	output: {
    path: path.resolve('build/js/'),
    publicPath: '/public/assets/js/',
    filename: 'bundle.js'
	},
  plugins: [
    // takes name of css file
    new ExtractTextPlugin("styles.css"),
    new webpack.ProvidePlugin({
      $: "jquery"
    }),
    // makes a json file that holds timestamp data
    // about when last time We run webpack
    new TimestampPlugin({
      // set location
      path: __dirname,
      // set name
      filename: 'timestamp.json'
    }),
    // adding banner to bundle.js file
    new webpack.BannerPlugin("|||||||||My Custom Banner|||||||||||\n")
  ],
  devServer: {
    contentBase: 'public'
  },
	module: {
		loaders: [
      {
        test: /\.json$/,
        exclude: /node_modules/,
        // puts path of custom loader that we created!
        loader: "json-loader!" + path.resolve('loader/strip')
      },
      {
  			test: /\.css$/,
        exclude: /node_modules/,
        loader: ExtractTextPlugin.extract( "style-loader", "css-loader")
      }
    ]
	}

}
```
