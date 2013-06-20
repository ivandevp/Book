# Node Twitter Clone
##### A simple twitter clone using Node.js and the Express web framework.

## Dependencies

To begin development of the twitter clone, you will need the following:

  - <a href="http://nodejs.org/" target="_blank">NodeJS</a>
  - <a href="http://git-scm.com/downloads" target="_blank">Git</a>
  
And get an account with each of the following services:

  - <a href="http://github.com" target="_blank">Github</a>
  - <a href="http://heroku.com" target="_blank">Heroku</a>

## Getting Started with NodeJS and NPM

First thing to do is to make sure you have node installed. On the command line, run the following:

`$ node --version`

You should then see a line similar to this:

`v0.10.2`

If you don't see that line, node is not installed. Go back to the installation section and make sure you have installed all the prerequisites.

Next, we are going to install the Express web framwork using NPM.

`$ npm install -g express`

You will start to see a lot of lines print out, basically this is the feedback npm outputs when installing a package. Don't worry too much about it for now. Again, let's check to make sure that it is installed properly.

```
$ express --version
3.2.6
```
Looks good. Now that we have Express installed, let's get started creating our project. Create a directory in your home folder called development.

`$ cd ~/`

then

`$ mkdir development`

Change directories into the development directory.

`$ cd development/`

This is the directory you will be using for development of apps on your machine. Within this directory, let's create a specific directory for our application.

`$ mkdir twitterclone`

then

`$ cd twitterclone/`

The `&&` operator will execute two commands in sequence. For example, the above two commands could be executed in one line as:

`$ mkdir twitterclone && cd twitterclone`

Using our favorite text editor, within our app directory we're going to create a package.json file that looks like this:

```
{
  "name": "twitterclone",
  "description": "Null to Node Twitter Clone App",
  "version": "0.0.1",
  "dependencies": {
    "express": "3.x"
  }
}
```

This file defines our express app and will tell npm what packages we will need to install. For now, we just have express listed as a dependency. The "3.x" means that our app requires any version of express, for example: `3.1`, `3.3.2` or `3.3.4`. We'll cover this in greater detail later on.

Now back to the command line, lets install our dependencies using npm. Within the development

`$ npm install`

This command tells npm to look within our current directory for a package.json file, and install the dependencies listed within it. Now if we list the contents of our directory we'll see a node_modules directory.

```bash
$ ls
node_modules  package.json
```

This node_modules directory contains all of the node libraries that our app will use. For now, don't worry about the contents of this directory, just make sure that it's there.

## Creating an Express Application

Now that we have express installed as a dependency within our app, we are ready to start building our application. To do so, lets create another file called app.js at the root of our project. In this case the root of our project is `~/development/twitterclone/`.

Okay, Inside of app.js lets write a couple lines to import express and initialize an app so that we can use it to serve up some content.

```javascript
var express = require('express');
var app = express();
```

Next, lets write our first route. In the following lines of code `'/hello'` is the path of our route. When a user requests the page /hello from within our app, this route will determine what we send back to them. In this case we are simply sending the text "Hello World" using `res.send('Hello World')`.

```javascript
app.get('/hello', function(req, res){
  res.send('Hello World');
});
```

Next, let's tell express to listen for requests on port 3000.

```javascript
app.listen(3000);
console.log('App running on port 3000');
```

Now, back to the command line, from within the root of our project run:

`$ node app.js`

You should see

`App running on port 3000`

Which means the app is running! Navigate to [http://localhost:3000/hello](http://localhost:3000/hello) to see what we've got so far, which should see a simple page showing the text "Hello World". Awesome! We just created a web server and served content via an express route.

To stop our node server; while in the command line where your app is running, hit ctrl+c. This is the command to kill a node process. After doing so, you should see your regular bash command prompt again. Now, if you navigate back to where our hello world page was it's no longer be available. To restart the app, just run `$node app.js` again. This process as a whole is how you restart a node application. Every time you make changes to your node application, you need to restart the server in order for those changes to be reflected.

### Separating our Routes

In app.js, we mapped a route `/hello` to call an anonymous function and that anonymous function served up some simple content. This works fine, but as our app grows, we probably aren't going to want all of our logic to be within our main app.js file. To seperate it up a bit and keep things a little more tidy, we're going to move our route functions to a new file called routes.js. So let's create a routes.js file in the same directory as app.js.

Now in app.js, lets import our routes file so that we can use it. At the top of our file, add the following line

```javascript
var routes = require('./routes');
```

Then, let's change our hello url mapping to one that will serve up a route in our routes file. Change this

```javascript
app.get('/hello', function(req, res){
  res.send('Hello World');
});
```

To this

```javascript
app.get('/', routes.welcome);
```

So now we're telling our app that when someone visits our site at the root path, we want to serve up a route called welcome that exists within routes. So let's go into our routes.js file and create the welcome route.

```javascript
exports.welcome = function(req, res) {
  res.send("Welcome");
}
```

When we called `require('./routes')` We are importing the routes.js file, and that file will give us access to the exports object. That is why we create our welcome function as a property of exports using `exports.welcome`. That means that when we import this file we will have access to it as a property called welcome. Anything else defined in this file will be unavailable outside of the file unless it is attached to the exports object. 

Now, let's restart the app and visit [http://localhost:3000/](http://localhost:3000/) and we should see our welcome page.

## Giving Our Site Some Structure

When a user comes to our site for the first time, we want to greet them with a welcome page where they can sign up for our site. In the this section we are going to create a welcome route and serve up our welcome page via using a template. To do this we are going to install a templating engine called handlebars. Template engines allow us to easily render dynamic content in our HTML files. Handlebars is a popular JavaScript templating engine and is quite widely used for node development. Let's start by adding it to our package.json file.

```javascript
{
  "name": "twitterclone",
  "description": "Null to Node Twitter Clone App",
  "version": "0.0.1",
  "dependencies": {
    "express": "3.x",
    "express3-handlebars": "0.4.x"
  }
}
```

Make sure to add a commas at the end of each dependecy EXCEPT for the last one or you will run into trouble. Now let's run npm install in our root directory so that it will install handlebars.

```bash
$ npm install
```

Now in app.js, let's import handlebars.

```javascript
var hbs = require('express3-handlebars');
```

And we need to tell our app to set handlebars as our templating engine.

```javascript
// Set handlebars as the default tempating engine
// and use main.handlebars as our default layout.
app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');
```

For reference, your appl.js file should look something like this now.

```javascript
var express = require('express');
var app = express();
var routes = require('./routes');
var hbs = require('express3-handlebars');

// Set handlebars as the default tempating engine
// and use main.handlebars as our default layout.
app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');

app.get('/', routes.index);

app.listen(3000);
console.log('App running on port 3000');
```

Now, since we are telling our app to use a layout file called main, we need to create it. Create a directory at the same level as app.js called views. Within that directory, create another directory called layouts. This will be where we keep all of our handlebars templates and layouts.

A layout is an html file that contains the structure of our site, it contains all of the content in our site that will persist from page to page, that way we don't have to write the same html files over and over. This is a really important concept to understand because it saves a lot of time down the road. If we want to change something with the layout of our site, we only have to change it in our layout file, instead of tediously updating every html file by hand and making mistakes along the way.

So let's create a really basic layout. Inside the layout directory let's create a file called main.handlebars and fill it with some basic HTML content.

```html
<!doctype html>
<html>
<head>
  <title>Twitter Clone</title>
</head>
<body>
  {{{ body }}}
</body>
</html>
```

Notice, that we have an odd line containing `{{{ body }}}`. This is the handlebars way of declaring that this is where we want the content of our template to be rendered. When we create a template, all of the content we put inside that template will be rendered where `{{{ body }}}` is in our layout. Very handy.

Now let's get to that template content. In our views directory let's create a file called welcome.handlebars and add a simple bit of HTML content.

```html
<h1>{{ title }}</h1>
<p>Glad you could make it!</p>
```

Create a welcome page using bootstrap


For simplicity, we're going to use Twitter's <a href="http://twitter.github.io/bootstrap/" target="_blank">Bootstrap</a> to develop the front end of our website. Bootstrap is a front-end framework that comes with some really nice features. It will help us create an app that looks good without much effort.

<a href="http://twitter.github.io/bootstrap/assets/bootstrap.zip">Download</a> the Bootstrap source files. Create a directory in the root of your project named public. Unzip the files and copy the bootstrap directory into /public.





