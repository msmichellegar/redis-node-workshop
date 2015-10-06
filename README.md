# An Introductory Redis Workshop

This is a workshop covering how to use Redis with Node.js.

To start with, if you'd like to read about *why* to use Redis, take a look at [this tutorial by DWYL](https://github.com/dwyl/learn-redis). You might also want to take a quick look at the [Redis command cheat sheet](https://github.com/FAC6/book/blob/master/patterns/week4/redisCheatsheet.md), and have a play yourself in [Try Redis](http://try.redis.io/).

It's a good idea to have Redis installed in your command line, which you can do easily with Homebrew. *For more installation instructions, look [here](http://redis.io/topics/quickstart).*

`brew install redis`

## The Challenge

In this workshop, we'll be building a simple list app. You'll be able to add list items to your database with an input box on the frontend. And you'll also see a list of all items in the database. Basically, the frontend should look a bit like this:

<hr>

![image of site](https://cloud.githubusercontent.com/assets/10683087/10299410/af74aafc-6be1-11e5-87d9-eff33e30cc34.png)

## Building the App

#### 1. Initialise the project

Create a repo. Run `npm init`. Make sure you have a `package.json`.

Establish some folder structure. Create a `public` folder to hold your `index.html` and your frontend JavaScript. Create a `server.js` file in your root.

#### 2. Create your `index.html` file

Nothing too fancy. The body should look something like this:

```html
<h5>My Favourite Things</h5>
<form>
    <input type="text" name="item"></input>
    <input type="submit"></input>
</form>
<ul class="favourite-things">
</ul>
```

#### 3. Set up your server

You'll be needing a basic Node server. Set one up in your `server.js`. If you need to recall how to build a server, look [here](https://github.com/nikhilaravi/learn-node).

Use a generic handler to serve the index.html file. Get it to appear in your browser.

#### 4. Set up your endpoints

You could write your app using different endpoints for your various actions (e.g. '/add' when the submit button is pressed, and '/display' when the page is loaded).  But remember [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete)!

You can write all your requests via one endpoint, namely '/'.  You will want to think about using 'POST' and 'GET' in your requests to differentiate.

If you were to use different endpoints instead of one, handlers might look something like this:

```js
var fs = require('fs');
var index = fs.readFileSync(__dirname + '/index.html');

function handler(request, response) {
    if (request.url.length === 1) {
        response.writeHead(200, {
            "Content-Type": "text/html"
        });
        response.end(index);
    } else if (request.url.indexOf('/add') > -1) {
        console.log("adding");
    } else if (request.url.indexOf('/display') > -1) {
        console.log("displaying");
    } else {
        fs.readFile(__dirname + request.url, function(err, file) {
            if (err) {
                response.writeHead(404, {
                    'Content-Type': 'text/' + ext
                });
                response.end();
            } else {
                var ext = request.url.split('.')[1];
                response.writeHead(200, {
                    'Content-Type': 'text/' + ext
                });
                response.end(file);
            }
        });
    }
};
```

Try to figure out how to write your server using only the '/' endpoint.

#### 5. Set up AJAX or HTTP requests on the frontend

You will need several requests: one for when you add an item to the database and one for displaying the data. The first should be triggered when you click the "submit" button. The second should be triggered on loading the page.

Luckily, there's a way of hooking up forms to make requests to the server. You just need to add the following method and action to the opening form tag: `method="POST" action="/"`.

Action is where you specify the endpoint that the data is destined for.


Now you've got the basic infrastructure set up, you're ready to go. Let's integrate Redis.

#### 6. Install `redis`

This is the go-to npm package for working with Redis.

`npm install redis --save`

#### 7. Start a Redis server

You'll need to start a Redis server. Run this command in your project folder. You should see a picture of a database.

``` $ redis-server ```

In another terminal window, check you can interact with the Redis client. After running these two commands, the terminal should respond with `empty list or set`.

```
$ redis-cli
$ keys *
```

#### 8. Hook up your code to Redis

First thing's first, require Redis and create a client.

```js
var redis  = require("redis");
var client = redis.createClient();
```

Now, instead of console-logging when you hit your endpoints, you can do something more productive like add to your database. The npm module has built-in methods for interacting with your Redis database.

For example, to add to a list, the raw Redis command goes `RPUSH favourites "kittens"`. The adapted Node version looks like this:

```js
client.rpush("favourites", "kittens", function(err, reply) {
    if (err) {
        console.log(err);
    } else {
        console.log(reply);
    }
})
```

It's important to note that every method takes a callback function as the final parameter. This will give you either an error or the response from your Redis client. If you're just adding data, the response won't be too useful. But if you're *fetching* data, this is where you'll find it.

You'll need to put these methods inside your handlers. Write a Redis method adding the item to a Redis list (you'll have to get this data from the frontend, take a look in the request object as it should send in the payload from the form). For displaying, write a Redis method fetching all the data from your database list.

*Look at [the documentation for the npm module](https://github.com/NodeRedis/node_redis). It's unfortunately a little vague, but you can at least guess at how most methods work.*

#### 9. Frontend JavaScript

Finally, to display your data on the frontend, you'll need a function that appends the data that comes back from your request to the `<ul>` list.

### You did it!

<hr>

For examples of Redis used in Node, take a look at these projects:
* [Best To Do Ever](https://github.com/msmichellegar/best-todo-ever)
* [DWYL's Tudo](https://github.com/dwyl/tudo)
* [JustOpine](https://github.com/justopine/justopine)
