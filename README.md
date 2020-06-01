[![General Assembly Logo](https://camo.githubusercontent.com/1a91b05b8f4d44b5bbfb83abac2b0996d8e26c92/687474703a2f2f692e696d6775722e636f6d2f6b6538555354712e706e67)](https://generalassemb.ly/education/web-development-immersive)

# Building a MERN App

The MERN stands for Mongo, Express, React and Node and is an acronym used to
describe applications build with that stack of technologies. React and it's
surrounding ecosystem (like Redux and React-Router) are just for the front-end
of the application. If we want to persist data to a database or have any other
back-end logic we need to (1) provide a separate back-end and (2) get the
front-end and back-end to communicate somehow.

In this lesson, we'll discuss two options for building full stack applications
using React on the front-end and Express, Mongo and Node on the back-end.

## Objectives

By the end of this, developers should be able to:

* Describe the difference between single-server and multi-server configurations
  for an application built in the MERN stack
* Install and use `cors` to allow for Cross-Origin Resource Sharing between our
  front-end application and back-end API
* Build and deploy our front-end application
* Describe the process of connecting our front-end and back-end
* Set up our front-end development server to proxy requests to our back-end
  server
* Set up our back-end API to serve static `build` assets in production

## Introduction (10 min / 0:15)

To integrate React with a back-end framework (such as Express) we will need to
make a few decisions about the desired architecture of our application (how we
want to structure everything). The first primary decision to make is where we
want our front-end application to be served from. There are two primary options:

### Multi-Server Configuration

Our front-end application and back-end API will be completely separate, hosted
on separate servers even! They will each have their own Git repositories and
will be deployed independently to different servers. Our front-end will retrieve
data from our back-end (across separate domains) using AJAX.

#### Pros

* Separation of Concerns: Our application is more modular and our front-end and
  back-end developers can work on their parts of the application independent
  of each other.
* Specialized Configuration: Since our front-end and API will be hosted on
  separate servers, we can optimize server configuration for each one
  independently, allowing them to scale independently of each other and
  preventing them from competing for resources on the same server.

#### Cons

* Since requests will be sent across separate domains, we will have to configure
  CORS (Cross-Origin Resource Sharing) for the browser to allow our front-end to
  retrieve data from our back-end

### Single-Server Configuration

Our front-end application will be housed in the same repository as our back-end
code. They will be deployed together to a single server where our back-end will
be responsible for serving up our front-end application in addition to our API
data.

#### Pros

* Single Deployment: Since our front-end will be served up by our API, we only
  need to manage one deployment.
* Unified Codebase: Since our front-end and back-end will be housed in the same
  Git repo, we will always have a more full picture of the application than if
  they were in separate repos.
* No CORS: Since our front-end application will be making requests back to the
  same domain that served it up (its 'origin'), we do not need to configure
  CORS.

#### Cons

* We will have to modify our server to serve up both json data for the API
  endpoints and the assets of our front-end which could require additional
  configuration and cause the two to compete over server resources.

Today, we will focus on setting up and deploying our application on separate
servers. This is more common and more straightforward.

## Getting Started (20 min / 0:35)

Today, we will be building the GameLib.biz app. GameLib.biz is a place for users to track their video game library and which games they've completed. 
We will also be using a Mongoose / Express back-end to allow for users to save their games.

### Back-End Setup

1. First, clone down [the
   back-end](https://git.generalassemb.ly/SF-SEI/gamelib-api),
   install dependencies, and open in VS Code.

2. Run the seeds file to populate our MongoDB database with game data.

```bash
node seed.js
```

3. Start the server

```bash
npm start
```

> If you inspect `package.json`, you will see that this is an alias for `nodemon server.js` in the "scripts" object

4. Navigate to `localhost:3001/api/v1/games` to see the response from our server. You should see a message that says: "Incomplete games#index controller function". 

5. In `controllers/games.js` all CRUD functionality has been stubbed out but is currently incomplete. Complete the `index` function with the following:

```
const index = (req, res) => {
    db.Game.find({}, (err, foundGames) => {
        if (err) console.log('Error in games#index:', err)
        
+        if(!foundGames) return res.json({
+            message: 'No Games found in database.'
+        })
+
+        res.status(200).json({ games: foundGames });
    })
}
```

### Front-End Setup

1. In a ***separate terminal window***, clone down [the React Translator
   app](https://git.generalassemb.ly/dc-wdi-react-redux/react-translator)
2. checkout to the `mern-starter` branch, install dependencies, and open in VS
   Code.
3. Start the development server:

```bash
npm start
```

> If you inspect `package.json`, you will see that this is an alias for `react-scripts start`

3. Navigate to `localhost:3000` to explore the application

## Application Dive (10 min / 0:45)

With a partner, take 5 minutes look through our back-end and 5 minutes to look through the front-end projects. Try
to answer the following questions.

* What npm packages are we using?
* What are the properties for our model?
* What are the routes? What components do they render? What's the nested structure look like?
* What CRUD functionality is currently available to us on the back-end?
* Where is the front-end making calls to our back-end?

## Two-Server Architecture (30 min / 1:15)

Currently we are using this type of architecture. Our back-end is running on
`localhost:3001` while our front-end is running on `localhost:3000`. One way to
say this is that these applications have different "origins". One issue with
this is that our browser is not going to like requests from our front-end
(served by `localhost:3000`) to our back-end on `localhost:3001`. In the
application, navigate to the Saved Translations view and check the console. You
should see:

```
XMLHttpRequest cannot load http://localhost:3001/api/translations. No 'Access-
Control-Allow-Origin' header is present on the requested resource. Origin
'http://localhost:3000' is therefore not allowed access.
```

This is Chrome telling us that since our back end is running on a separate port
("origin") than our front end, our front end is not allowed to retrieve data
from it. To fix this, we need to configure CORS (Cross-Origin Resource Sharing)
on our back-end.

### Installing `cors` in Express

1. Stop your Express server, and install `cors` via npm:

```bash
npm install cors
```

2. In `index.js`, require the `cors` module and integrate with Express:

```js
const cors = require('cors')
```

```js
app.use(cors())
```

3. Restart your Express server:

```bash
npm start
```

Now when you navigate to `localhost:3000/translations`, you should see a list of
the translations being retrieved from our API.

> Note: The default `cors` configuration (above) will allow requests from
> **any** origin (which may or may not be ideal). To more precisely control
> access to our API, we would need to do little more configuration. Check out
> the [cors documentation](https://www.npmjs.com/package/cors) for more
> information on this process.

### Deploying to Separate Domains

Now that our front end and back end are communicating properly, let's look at
how we would deploy them.

#### Back End Deployment

As our back-end will need to be hosted on a server and host a database, we will
want to deploy it to a hosting service like Heroku.

We would go about this [no differently than we have
previously](https://git.generalassemb.ly/dc-wdi-node-express/heroku-mlab-deployment).

Check out a branch that has this set up:

```bash
git checkout microservice-solution
```

If you open the code, you'll see it has the `index.js` file set up with the port
environment variable and the `connection.js` file set up to connect with
a database.

Make sure you're logged into the heroku cli:

```bash
heroku login
```

Then create a new heroku application:

```bash
heroku create react-translator-api
```

> Note: `heroku create` creates the application and sets up the remote
> [(documentation on this)](https://devcenter.heroku.com/articles/creating-apps)

> If everyone runs this at the same time it may throw an error. Try adding your initials to the end to make it unique.

Next, we want to create a new database on [Mongo
Atlas](https://cloud.mongodb.com). Once you've created a new database, create
a user for that database. Then, in your terminal (inserting the data for your
Atlas db):

```bash
heroku config:set DB_URL="mongodb+srv://<username>:<password>@cluster0-ru2re.mongodb.net/test?retryWrites=true"
```

> Be sure to replace <username> and <password> with your database username and
> password

Finish setting up your database. Then, push our app to Heroku:

```bash
git push heroku microservice-solution:master
```

Then seed the database:

```bash
heroku run node db/seeds.js
```

#### Front-End Deployment

Now that our back-end is deployed to Heroku, we need to update our front-end with the correct URLs in our axios requests. Right now, axios is requesting data from our local server:

```js
axios.get('http://localhost:3001/api/translations')
```

Update your axios request with your Heroku URL and `'/api/transtations'` For example:

```js
axios.get('<insert Heroku URL here>/api/translations')
```
> To quickly get access to your back-end Heroku URL, run `heroku open` in the terminal of your back-end repo and copy the URL from your browser.

Currently, when we start our React application locally, we are using
`react-scripts`. `react-scripts` is the black box that contains all of the major
dependencies and configuration that we are using in development:

* Dependencies: Babel, Webpack, and ESLint and other dependencies that allow the
  use of JSX, hot reloading, compilation of our application, and other features.
* Configuration: Configuration files for the above dependencies that control
  (among other things) how our app is compiled in both development and
  production environment.
* Scripts: Commands that allow us to do things like start our Webpack server
  (`react-scripts start`) or create a compiled, minified version of our
  application (in vanilla JS) that can be deployed (`react-scripts build`).

> Source code on `react-scripts` [here](https://github.com/facebook/create-react-app/tree/master/packages/react-scripts).

The command that we will need to use for deploying our front-end is
`react-scripts build`. `create-react-app` automatically aliases this for us in
our `package.json` as `npm run build`. In the terminal, run this command to
create a deployment-ready version of your app:

```bash
npm run build
```

This will create a directory called `build` in the root of your application.
This directory contains all of your application's HTML, minified JS, and
minified CSS in a deployment-ready format.

Now all you would need to do is to deploy **the build directory** to a static
asset hosting service (such as [GitHub Pages](https://pages.github.com/) or
[surge](https://surge.sh/).

If pushing to gh-pages:

```bash
git subtree push --prefix build origin master:gh-pages
```

> Note that `subtree` allows us to only push a sub-directory of our repository
> instead of the entire app. [More
> Information](https://gist.github.com/cobyism/4730490)

> Pro move: create a `deploy` npm script for your deployment command.

If using surge:

```bash
npm install -g surge
surge ./build
```

> [Surge](https://surge.sh/) is a CLI based npm package that lets you quickly
> deploy static front-end applications for free.

## Closing / Questions (10 min / 2:30)

## Additional Resources

### Microservice Solution

This is the setup that we implemented together in class.

* Back-End: https://git.generalassemb.ly/cd-wdi-react-redux/react-translator-api/tree/microservice-solution
* Front-End: https://github.com/cd-wdi-react-redux/react-translator/tree/mern-starter

### Single-Server Solution

If you'd like to see an example of a single-server implementation, you can check
one out here:

* Back-End: https://git.generalassemb.ly/cd-wdi-react-redux/react-translator-api/tree/single-server-solution

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
