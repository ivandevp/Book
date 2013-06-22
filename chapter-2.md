# Chapter Two: Layouts and Templates

When a user comes to our site for the first time, we want to greet them with a welcome page where they can sign up for our site. In the this section we are going to create a welcome route and serve up our welcome page via using a template. To do this we are going to install a templating engine called handlebars. Template engines allow us to easily render dynamic content in our HTML files. Handlebars is a popular JavaScript templating engine and is quite widely used for node development. Let's start by adding it to our package.json file.

```javascript
{
  "name": "nulltonode",
  "description": "Null to Node Sample App",
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
var express = require('express');
var app = express();
var routes = require('./routes');
var hbs = require('express3-handlebars');

...
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

app.get('/', routes.home);

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

Now in our home route, let's tell the app to serve up a template called 'home' when the user visits the root path of our site. We do this by calling response.render and passing in the name of our template which is 'home'.

```javascript
exports.home = function (request, response) {
  response.render('home');
}
```

Let's restart our app and see what our welcome page looks like now.

<img src="http://cl.ly/image/3K023i3z0a3R/Screen%20Shot%202013-06-21%20at%2011.07.20%20AM.png">

Not too exciting, but that is about to change. We're going to make our layout a bit nicer. 

For simplicity, we're going to use Twitter's <a href="http://twitter.github.io/bootstrap/" target="_blank">Bootstrap</a> to develop the front end of our website. Bootstrap is a front-end framework that comes with some really nice features. It will help us create a nice clean site that looks good without much effort.

<a href="http://twitter.github.io/bootstrap/assets/bootstrap.zip">Download</a> the Bootstrap source files. Create a directory in the root of your project named public. Unzip the files and copy the bootstrap directory into /public. We also need to tell our app that we will be serving files up from our public directory.

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
            <li><a href="/">Home</a></li>
            <li><a href="/users">Users</a></li>
            <li class="dropdown">
              <a href="#" class="dropdown-toggle" data-toggle="dropdown">Account <b class="caret"></b></a>
              <ul class="dropdown-menu">
                <li><a href="/profile">My Profile</a></li>
                <li><a href="/settings/">Log out</a></li>
              </ul>
            </li>
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

This is a very basic bootstrap layout taken from the <a href="http://twitter.github.io/bootstrap/examples/starter-template.html">Bootstrap Starter Template</a>. We have a simple page layout containing a navbar with a home button and a navbar pulled to the right. At the end of our body element, we included a bootstrap.js and jQuery

```html
<script src="http://code.jquery.com/jquery-1.10.1.min.js"></script>
<script src="/public/bootstrap/js/bootstrap.js" type="text/javascript"></script>
```

Also, in the head of the file, we have included a the bootstrap CSS and our own layout.css which we will create next.

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

<img src="http://cl.ly/image/3w2U342M4546/Screen%20Shot%202013-06-21%20at%2011.10.40%20AM.png">

Now we've handled the basics of creating a layout and template. In the next chapter we're going to build upon this concept by adding some structure and navigation to our site.


