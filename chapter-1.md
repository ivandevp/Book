# Chapter One: Getting Started

## Installing Prerequisites

To begin development of the sample app we're going to be working on, you'll need the following:

  - <a href="http://nodejs.org/" target="_blank">NodeJS</a>: The server-side software for our project.
  - <a href="http://www.mongodb.org/downloads" target="_blank">MongoDB</a>: The database we will be using to store data.

If you have not done so already, download and install them.

### Checkpoint!

Let's see if Node installed properly. In the Command Line, run the following: (NOTE: The $ sign is simply a flag signifying the Command Line. Do not type it when running the command. For the rest of this tutorial, anytime you see $, know that what follows refers to the Command Line.)

`$ node --version`

You should then see a line similar to this:

`$ v5.4.1`

If you don't see that line, Node is not installed. Go back to the installation section and make sure you have installed all the prerequisites.

Next, let's check if Mongodb is installed. In the Command Line, run:

`$ mongod --version`

A line similar to the following should print out:

`$ db version v2.6.4`

If so, great! Let's move on. If you don't see that line, Mongodb is not installed. Click the link for Mongodb above and follow the installation instructions.

## Using NodeJS and NPM to install Express

Next, we are going to install the Express web framwork using NPM. NPM stands for Node Package Manager. It is a standard way to install dependencies into a development project.

`$ npm install -g express`

You will start to see a lot of lines print out, basically this is the feedback npm outputs when installing a package. Don't worry too much about it for now.

If you get errors at this stage you probably need to install Express as the administrator. Use this command. You will be prompted for your adminstrator password. (NOTE: Often the Command Line will not print anything as you type your password. This is a security feature. Type your password and press enter and the command should run.)

`$ sudo npm install -g express`

### Checkpoint!

Again, let's check to make sure that it is installed properly.

```
$ express --version
4.9.0
```

If anything other than the version number printed out, you will need to troubleshoot. Helping with every possible error is outside the scope of this tutorial, so you will need to consult Google.

Most likely things worked, however, and Express is now installed. Let's get started creating our project. 

## Using Command Line

We will be using Command Line to create and navigate through files and folders. Here is a <a href="https://gist.github.com/thom801/5664013">cheatsheet</a> for further reference.

Create a directory in your home folder called development.

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

Using your favorite text editor, within our app directory we're going to create a package.json file that looks like this:

```
{
  "name": "nulltonode",
  "description": "Null to Node Sample App",
  "version": "0.0.1",
  "dependencies": {
    "express": "4.x"
  }
}
```

This file defines our express app and will tell npm what packages we will need to install. For now, we just have express listed as a dependency. The "3.x" means that our app requires any version of express, for example: `4.1`, `4.3.2` or `4.8.4`. We'll cover this in greater detail later on.

Now back to the command line, let's install our dependencies using npm. Within the development directory run:

`$ npm install`

This command tells npm to look within our current directory for a package.json file, and install the dependencies listed within it.

###Checkpoint!

Now if we list the contents of our directory we'll see a node_modules directory.

```bash
$ ls
node_modules  package.json
```

This node_modules directory contains all of the node libraries that our app will use. For now, don't worry about the contents of this directory, just make sure that it's there.

## Creating an Express Application

Now that we have express installed as a dependency within our app, we are ready to start building our application. To do so, let's create another file called app.js at the root of our project. In this case the root of our project is `~/development/nulltonode/`.

Okay, Inside of app.js lets write a couple lines to import express and initialize an app so that we can use it to serve up some content.

```javascript
var express = require('express');
var app = express();
```

`require` is a NodeJS specific function used to load a package into your code. It will look inside the `node_modules` directory and find the named package. In this case, `express`. In the second line we call `express` and store it as the variable `app`. From now on, when we type any line of code prefaced with `app`, it means we are using the `express` framework to accomplish something.

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
###Checkpoint!

Now, back to the command line, from within the root of our project run:

`$ node app.js`

You should see

`App running on port 3000`

Which means the app is running! Navigate a browser to [http://localhost:3000/hello](http://localhost:3000/hello) to see what we've got so far, which should see a simple page showing the text "Hello World". Awesome! We just created a web server and served content via an express route.

To stop our node server; while in the command line where your app is running, hit ctrl+c. This is the command to kill a node process. After doing so, you should see your regular bash command prompt again. Now, if you navigate back to where our hello world page was and refresh the page, you will see that it's no longer available. To restart the app, just run `$ node app.js` again. (Press the up arrow while in command line to get the last command you ran. Then press return.) This process as a whole is how you restart a node application. NOTE: Every time you make changes to your node application, you need to restart the server in order for those changes to be reflected.

## Including Routes

In app.js, we mapped a route `/hello` to call an anonymous function and that anonymous function served up some simple content. This works fine, but as our app grows, we probably aren't going to want all of our logic to be within our main app.js file. To separate it up a bit and keep things a little more tidy, we're going to move our route functions to a new file called routes.js. So let's create a routes.js file in the same directory as app.js.

Now in app.js, lets import our routes file so that we can use it. At the top of our file, below `var app = express();` add the following line

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

When we called `require('./routes')` we are importing the routes.js file, and that file will give us access to the exports object. That is why we create our home function as a property of exports using `exports.home`. That means that when we import this file we will have access to it as a property called home. Anything else defined in this file will be unavailable outside of the file unless it is attached to the exports object.

###Checkpoint!

Now, let's restart the app (See the last checkpoint if you don't remember how.) and visit [http://localhost:3000/](http://localhost:3000/) and we should see our welcome page.

[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-2.md)
