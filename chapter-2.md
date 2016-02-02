# Chapter Two: Layouts and Templates

## Installing Our Templating Engine

When a user comes to our site for the first time, we want to greet them with a welcome page where they can sign up for our site. In this section we're going to create a welcome route and serve up our welcome page using a template. To do this we're going to install a templating engine called handlebars. Template engines allow us to easily render dynamic content in our HTML files. Handlebars is a popular JavaScript templating engine and is quite widely used for node development. Let's start by adding it to our package.json file.

```javascript
{
  "name": "nulltonode",
  "description": "Null to Node Sample App",
  "version": "0.0.1",
  "dependencies": {
    "express": "4.x",
    "express-handlebars": "3.x"
  }
}
```

Make sure to add a comma at the end of each dependency EXCEPT for the last one or you will run into trouble. Now let's run npm install in our root directory so that it will install handlebars.

```bash
$ npm install
```

Now in app.js, let's import handlebars.

```javascript
var express = require('express');
var app = express();
var routes = require('./routes');
var hbs = require('express-handlebars');

...
```

And we need to tell our app to set handlebars as our templating engine.

```javascript
// Set handlebars as the default templating engine
// and use main.handlebars as our default layout.
app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');
```
### Checkpoint!

Your app.js file should look something like this now.

```javascript
var express = require('express');
var app = express();
var routes = require('./routes');
var hbs = require('express3-handlebars');

// Set handlebars as the default templating engine
// and use main.handlebars as our default layout.
app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');

app.get('/', routes.home);

app.listen(3000);
console.log('App running on port 3000');
```

## Creating a Template

Now, since we are telling our app to use a layout file called main, we need to create it. Create a directory at the same level as app.js called views. This is where we will keep all of our handlebars templates and layouts. Within the views directory, create another directory called layouts. This directory structure is required for Handlebars to find our layout file. To find out more, visit [here](https://github.com/ericf/express3-handlebars#basic-usage).

A layout is an html file that contains the structure of our site.  It contains all of the content in our site that will persist from page to page so we don't have to write the same html files over and over. This is a really important concept to understand because it saves a lot of time down the road. If we want to change something with the layout of our site, we only have to change it in our layout file, instead of tediously updating every html file by hand and making mistakes along the way.

So let's create a really basic layout. Inside the layouts directory let's create a file called main.handlebars and fill it with some basic HTML content.

```html
<!doctype html>
<html>
<head>
  <title>Null to Node</title>
</head>
<body>
  {{{ body }}}
</body>
</html>
```

Notice, that we have an odd line containing `{{{ body }}}`. This is the handlebars way of declaring that this is where we want the content of our template to be rendered. When we create a template, all of the content we put inside that template will be rendered where `{{{ body }}}` is in our layout. Very handy.

Now let's get to that template content. In our views directory let's create a file called home.handlebars and add a simple bit of HTML content.

```html
<h1>Welcome</h1>
<p>Glad you could make it!</p>
```

Now in our routes.js file in the home route, let's tell the app to serve up a template called 'home' when the user visits the root path of our site. We do this by calling response.render and passing in the name of our template which is 'home'. Go ahead and remove the response.send("Welcome"); line and replace it with response.render('home'); as shown below.

```javascript
exports.home = function (request, response) {
  response.render('home');
}
```

`render` is an ExpressJS method. Because we registered Handlebars as the templating engine with Express in the app.js file, the `render` method knows we want the `home.handlebars` template.

### Checkpoint!

Let's restart our app and see what our welcome page looks like now. It should look like this:

<img src="http://cl.ly/image/3K023i3z0a3R/Screen%20Shot%202013-06-21%20at%2011.07.20%20AM.png">

Not too exciting, but that's about to change. We're going to make our layout a bit nicer.

## Styling the Template

For simplicity, we're going to use Twitter's <a href="http://getbootstrap.com/2.3.2/" target="_blank">Bootstrap</a> to develop the front end of our website. Bootstrap is a front-end framework that comes with some really nice features. It will help us create a nice clean site that looks good without much effort.

<a href="http://getbootstrap.com/2.3.2/">Download</a> the Bootstrap source files. Create a directory in the root of your project named public. Unzip the files and copy the bootstrap directory into /public. We also need to tell our app that we will be serving files up from our public directory.

```javascript
...

app.engine('handlebars', hbs({defaultLayout: 'main'}));
app.set('view engine', 'handlebars');
app.use('/public', express.static('public'));

...
```

Now let's modify our main.handlebars file to look like this.

```html
<!doctype html>
<html>
<head>
  <title>Twitter Clone</title>
  <link href="/public/bootstrap/css/bootstrap.css" rel="stylesheet">
  <link href="/public/stylesheets/layout.css" rel="stylesheet">
</head>
<body>

  <div class="navbar navbar-inverse navbar-fixed-top">
    <div class="navbar-inner">
      <div class="container">
        <a class="brand" href="/">Null to Node</a>
        <div class="nav-collapse collapse">
          <ul class="nav pull-right">
            <li><a href="/login">Log In</a></li>
          </ul>
        </div>
      </div>
    </div>
  </div>

  <div class="container">
    {{{ body }}}
  </div>

  <script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
  <script src="/public/bootstrap/js/bootstrap.js" type="text/javascript"></script>
</body>
</html>
```

This is a very basic bootstrap layout taken from the <a href="http://twitter.github.io/bootstrap/examples/starter-template.html">Bootstrap Starter Template</a>. We have a simple page layout containing a navbar with a home button and a navbar pulled to the right. At the end of our body element, we included bootstrap.js and jQuery with script tags.

```html
<script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
<script src="/public/bootstrap/js/bootstrap.js" type="text/javascript"></script>
```

Also, in the head of the file, we have included the bootstrap CSS and our own layout.css which we will create next.

```html
<link href="/public/bootstrap/css/bootstrap.css" rel="stylesheet">
<link href="/public/stylesheets/layout.css" rel="stylesheet">
```

Let's do that now. Inside your public directory, create a new directory and name it stylesheets. Within that directory create a file named layout.css. Within layout CSS add the following style definition.

```css
body {
  padding-top: 60px;
}

.container {
  max-width: 600px;
}
```

Pretty simple for now. We're just adding a bit of padding to the body of our layout so that our content doesn't show up underneath the navigation. Now, let's restart our server and take a look at our home page.

### Checkpoint!

Does your site look like this?

<img src="http://cl.ly/image/1Y1L2P2y441t/Screen%20Shot%202013-07-29%20at%203.57.16%20PM.png">

Basically, we created a page layout (main.handlebars), and then a view (home.handlebars), that we inject inside of main.handlebars where the handlebars synax (`{{{ body }}}`).

## Creating the Sign Up Page

Obviously we want to do more than just greet our users. Let's give them the ability to sign up to use our application. To do this we will build a basic HTML form. Change home.handlebars to this:

```html
<h1>Welcome</h1>
<p>To get started, fill out the sign up form below.</p>
<hr>
<p class="lead">Sign Up</p>
{{#if formError }}
  <p class="alert alert-warning">{{{ formError }}}</p>
{{/if}}
<form action="" method="POST">
  <input type="text" name="name" value="" placeholder="Name"><br/>
  <input type="text" name="username" value="" placeholder="username"><br/>
  <input type="password" name="password" value="" placeholder="password"><br/>
  <p>Profile Image</p><input type="file" name="image"><hr>
  <input class="btn btn-primary" type="submit" name="" value="Sign Up">
</form>
```

In this form, we have an input for name, username, and password along with a file input for the user's profile image. Notice that the form method is set to `POST`, this will tell our form to post data to the action url when the form is submitted. In this case our action is blank which will post to the current location. Another thing to note is `enctype="multipart/form-data"` which will ensure that our image file will be posted to the server.

If the form submission returns an error, the error will be shown above the form.

```html
{{#if formError }}
  <p class="alert alert-warning">{{{ formError }}}</p>
{{/if}}
```

### Checkpoint!

After restarting node and refreshing your page, you should now see a basic signup form. Congrats, we have the first page of our application!

Now we've handled the basics of creating a layout and template. In the next chapter we're going to build upon this by creating the functionality to actually save a user's sign up information.

[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-3.md)
