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

Our route is calling response.render, which tells the app to look for whatever template name we pass in as a parameter. In this case we are asking the app to render a template called 'users' which doesn't exist yet. Let's create that now. Within the views directory, create a new file called users.handlebars and type the following code into that file.

```html
<h1>Users</h1>
<p>Here's where we'll list out our users.</p>
```
