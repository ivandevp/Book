# Chapter Three: Site Structure and Routing

## Navigation with Routes

In this section, we are going to create some routes in our routes.js file that will serve up different pages. Right now, if you click users or account on the site navigation, you'll see a page that reads "Cannot get /users" etc. What we want to do is make our navigation work, so that when you visit /users, you will see a users page and so on. Let's get started by telling our app that when the user requests the path /users to call a route called users in our routes file.

```javascript
app.get('/', routes.home);
app.get('/users', routes.users);
```

Now let's create that new users route in the routes.js file.

```javascript
exports.users = function (request, response) {
  response.render('users');
}
```

Our route is calling response.render, which tells the app to render a template. In this case we are asking the app to render a template called 'users' which doesn't exist yet. Let's create that now. Within the views directory, create a new file called users.handlebars and type the following code into that file.

```html
<h1>Users</h1>
<p>Here's where we'll list out our users.</p>
```

So now we have url and route defined for users. Next we need to do the same for our account links. If you click on the Account dropdown, you'll see we have two links, one to view your profile page which links to /profile, and a Log Out button that links to /logout. Let's start by defining these urls in our app.js file.

```javascript
app.get('/profile', routes.profile);
app.get('/logout', routes.logout);
```

And again, lets add the corresponding routes to our routes.js file. We'll just put a TODO comment in our logout route so that we remember to build that functionality later on.

```javascript
exports.profile = function (request, response) {
  response.render('profile');
}

exports.logout = function (request, response) {
  // TODO: Build logout functionality
}
```

Now, we need to create our profile template.


[Next Chapter >>](https://github.com/NullToNode/Book/blob/master/chapter-4.md)
