# Chapter One: Getting Started

## What you'll need

To begin development of the sample app we're going to be working on, you'll need the following:

  - <a href="http://nodejs.org/" target="_blank">NodeJS</a>: The server-side software for our project.
  - <a href="http://www.mongodb.org/downloads" target="_blank">MongoDB</a>: The database we will be using to store data.

## NodeJS and NPM

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

`$ mkdir nulltonode`

then

`$ cd nulltonode/`

The `&&` operator will execute two commands in sequence. For example, the above two commands could be executed in one line as:

`$ mkdir nulltonode && cd nulltonode`

Using our favorite text editor, within our app directory we're going to create a package.json file that looks like this:

```
{
  "name": "nulltonode",
  "description": "Null to Node Sample App",
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

Now that we have express installed as a dependency within our app, we are ready to start building our application. To do so, lets create another file called app.js at the root of our project. In this case the root of our project is `~/development/nulltonode/`.

Okay, Inside of app.js lets write a couple lines to import express and initialize an app so that we can use it to serve up some content.

```javascript
var express = require('express');
var app = express();
```

Next, lets write our first route. In the following lines of code `'/hello'` is the path of our route. When a user requests the page /hello from within our app, this route will determine what we send back to them. In this case we are simply sending the text "Hello World" using `res.send('Hello World')`.

```javascript
app.get('/hello', function (request, response){
  response.send('Hello World');
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

## Including Routes

In app.js, we mapped a route `/hello` to call an anonymous function and that anonymous function served up some simple content. This works fine, but as our app grows, we probably aren't going to want all of our logic to be within our main app.js file. To seperate it up a bit and keep things a little more tidy, we're going to move our route functions to a new file called routes.js. So let's create a routes.js file in the same directory as app.js.

Now in app.js, lets import our routes file so that we can use it. At the top of our file, add the following line

```javascript
var routes = require('./routes');
```

Then, let's change our hello url mapping to one that will serve up a route in our routes file. Change this

```javascript
app.get('/hello', function (request, response){
  response.send('Hello World');
});
```

To this

```javascript
app.get('/', routes.home);
```

So now we're telling our app that when someone visits our site at the root path, we want to serve up a route called home that exists within routes. So let's go into our routes.js file and create the home route.

```javascript
exports.home = function (request, response) {
  response.send("Welcome");
}
```

When we called `require('./routes')` we are importing the routes.js file, and that file will give us access to the exports object. That is why we create our welcome function as a property of exports using `exports.welcome`. That means that when we import this file we will have access to it as a property called welcome. Anything else defined in this file will be unavailable outside of the file unless it is attached to the exports object. 

Now, let's restart the app and visit [http://localhost:3000/](http://localhost:3000/) and we should see our welcome page.

[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-2.md)
