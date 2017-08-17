---
title: Movies Example
---

Out of the box, Vulcan comes with many features such as handling posts and comments, generating a newsletter, and much more. 

But it's important to understand that you can also use the underlying framework powering these features directly and build a completely different type of app. 

So in this tutorial, we'll focus on understanding this framework and seeing how to build a simple paginated list of movies. 

The completed code for this tutorial can be found in [the `example-movies` package](https://github.com/TelescopeJS/Telescope/tree/master/packages/example-movies).

## Video Tutorial

<iframe width="100%" height="315" src="https://www.youtube.com/embed/4HidaFce6j0" frameborder="0" allowfullscreen></iframe>

Note: this video differs slightly from the following tutorial when it comes to [defining the `user` resolver on the `userId` field](#The-User-Resolver). Both approaches will work, but make sure you check the tutorial to get the most up-to-date syntax. 

## Short & Long Versions

This tutorial includes a shorter version that uses some of Vulcan's default presets to save you time. If you'd rather to this short version, just skip every section marked with a `*` sign.

## Set Up

First, make sure you're only including the packages you need. This means everything in your `.meteor/packages` file should be commented out except for the core packages, a language package, and an accounts package.

Here's a quick overview of the main packages:

- `vulcan:core`: Vulcan's core libraries. 
- `vulcan:forms`: the library used for generating and submitting forms. 
- `vulcan:routing`: sets up and initializes routing and server-side rendering.
- `vulcan:users`: user management (groups, permissions, etc.).

We'll also keep `vulcan:i18n-en-us`, since it contains strings for core UI features, as well as Meteor's `accounts-password` package to activate password log in.

## Creating a Package

The next step will be creating a Meteor package to hold our code. This will be a *local* package, meaning it will live inside your repo, but we'll still have all the advantages of regular, remote packages (easy to enable/disable, restricted scope, ability to specify dependencies, etc.).

Create a new `my-package` directory under `/packages`. We'll use the following file structure:

```
my-package
  lib
    client
      - main.js
    components
      movies
    modules
      movies
      - index.js
    server
      - main.js
    stylesheets
  - package.js
```

- `package.js` is your [package manifest](http://docs.meteor.com/api/packagejs.html), and it tells Meteor which files to load.
- `client/main.js` and `server/main.js` are the client and server entry points.
- `modules/index.js` lists all the various modules that make up our package. 
- `style.css` contains all our styles.

Set up all the directories along with the blank files within. Once this is done, let's start with the `package.js` file:

```js
Package.describe({
  name: 'my-package',
});

Package.onUse(function (api) {

  api.use([
    'vulcan:core',
    'vulcan:forms',
    'vulcan:accounts',
  ]);

  api.addFiles('lib/stylesheets/bootstrap.min.css', 'client');

  api.mainModule('lib/server/main.js', 'server');
  api.mainModule('lib/client/main.js', 'client');

});
```

The `api.use` block defines our package's dependencies: `vulcan:core`, `vulcan:forms`, and `vulcan:accounts`, a package that provides a set of React components for managing log in and sign up. 

We then define our two client and server endpoints using `api.mainModule`. Create `lib/client/main.js` and `lib/server/main.js`, and paste in the following line in both:

```js
import '../modules/index.js';
```

Finally, let's include the [Bootstrap](http://getbootstrap.com) stylesheet in `lib/stylesheets/bootstrap.min.css` to make our app look a little better. You can get it from [here](https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.4/css/bootstrap.min.css).

The last step to activate your package is enabling it using the `meteor add` command to add it to your `.meteor/packages` file:

```
meteor add my-package
```

It may seem like not much happened, but once Meteor restart our custom package will now be active and loaded. 

## Our First Component

We now have the basic structure of our package, so let's get to work. We'll create a new component and a new route to display it. 

First, create a new `components` directory inside `lib` if you haven't done so yet and add the `movies` directory. Inside `components/movies` add a new file containing a `MoviesList` component:

```js
import React, { PropTypes, Component } from 'react';
import { Components, withList, withCurrentUser, Loading } from 'meteor/vulcan:core';

const MoviesList = ({results = [], currentUser, loading, loadMore, count, totalCount}) => 
  
  <div style={ { maxWidth: '500px', margin: '20px auto' } }>

    Hello World!

    {/* user accounts placeholder */}

      <div className="movies">
        
        {/* new document form placeholder */}
        
        {/* documents list placeholder */}
        
        {/* load more placeholder */}

      </div>

  </div>

export default MoviesList;
```

## Routing

We can now create a [route](/routing.html) to display this component. Create a new `routes.js` file inside `modules`:

```js
import { addRoute } from 'meteor/vulcan:core';

import MoviesList from '../components/movies/MoviesList.jsx';

addRoute({ name: 'movies', path: '/', component: MoviesList });
```

In case you're wondering, `addRoute` is a very thin wrapper over [React Router](https://github.com/ReactTraining/react-router). 

Make sure to also import `routes.js` inside `modules/index.js`:

```js
import './routes.js';
```

If everything worked properly, you should now be able to head to `http://localhost:3000/` and see your `MoviesList` component show up. 

## The Schema

We want to display a list of movies, which means querying for data as well as setting up basic insert, edit, and remove operations. But before we can do any of that, we need to define what a “movie” is. In other words, we need a schema. 

Vulcan uses JSON schemas based on the [SimpleSchema](https://github.com/aldeed/meteor-simple-schema) package. You can also check out the [Collections & Schemas](/schemas.html) section if you want to learn more. 

Create `schema.js` inside `modules/movies`:

```js
const schema = {

  // default properties

  _id: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  createdAt: {
    type: Date,
    optional: true,
    viewableBy: ['guests'],
    onInsert: () => {
      return new Date();
    },
  },
  userId: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  
  // custom properties

  name: {
    label: 'Name',
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  review: {
    label: 'Review',
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },

};

export default schema;
```

Note that we're setting up an `onInsert` function on the `createdAt` field to initialize it to the current timestamp whenever a new document is inserted. 

And we're also setting `viewableBy: ['guests']` on every field to make sure they're visible to non-logged-in users (who belong to the default `guests` group). By default, any schema field is kept private, so we need to make sure we don't forget this step if we want our data to be publicly accessible.

## Setting Up a Collection

We're now ready to set up our `Movies` collection. Create a new `collection.js` file inside `modules/movies`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';

const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,

});

export default Movies;
```

`createCollection` takes in a collection name, type name (in other words, the GraphQL type of documents belonging to this collection), a schema (as well as a few other objecs we'll learn about later) and sets up a fully working GraphQL data layer for you!

As we did previously with `routes.js`, import `collection.js` from `index.js`:

```js
// The Movies collection
import './movies/collection.js';

// Routes
import './routes.js';
```

At this point it might not look like much has changed, but we now have a functional GraphQL schema! You can see it by opening up the Meteor shell in a terminal window (by typing `meteor shell` from within your app directory) and typing:

```
import {GraphQLSchema} from 'meteor/vulcan:lib'
GraphQLSchema.finalSchema
```

But even though we have a schema, we can't actually query for a document yet because we don't have any **query resolvers**. 

## Note: Default Resolvers & Mutations

Vulcan provides a set of [default resolvers](/data-loading.html#Default-Resolvers) to speed things up, but for the sake of learning how things work this tutorial will show you how to write resolvers yourself. 

If you'd rather skip this and use the defaults for now, just use the following code in `collection.js`, and simply ignore any section concerning the `resolvers.js` or `mutations.js` files (they will be marked with a `*` sign):

```js
import { createCollection, getDefaultResolvers, getDefaultMutations } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import './fragments.js';
import mutations from './mutations.js';
import './permissions.js';
import './parameters.js';

const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,
  
  resolvers: getDefaultResolvers('Movies'),

  mutations: getDefaultMutations('Movies'),

});

export default Movies;
```

## Custom Query Resolvers*

*(Note: you can skip this section if you're using default resolvers and mutations)*

In a nutshell, a resolver tells the server how to respond to a specific GraphQL query, and you can learn more about them in the [Data Loading](/data-loading.html) section. 

Let's start simple. Create a new `resolvers.js` file inside `modules/movies` and add:

```js
const resolvers = {

  list: {

    name: 'moviesList',

    resolver(root, args, context) {
      return context.Movies.find().fetch();
    },

  },

};

export default resolvers;
```

Then import this new file from `collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';

const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

});

export default Movies;
```

## Seeding The Database

We can try out our new query resolver using [GraphiQL](https://github.com/graphql/graphiql), but first we need some data. Create a new `seed.js` file inside `server`:

```js
import Movies from '../modules/movies/collection.js';
import Users from 'meteor/vulcan:users';
import { newMutation } from 'meteor/vulcan:core';

const seedData = [
  {
    name: 'Star Wars',
    year: '1973',
    review: `A classic.`,
    privateComments: `Actually, I don't really like Star Wars…`,
  },
  {
    name: 'Die Hard',
    year: '1987',
    review: `A must-see if you like action movies.`,
    privateComments: `I love Bruce Willis so much!`,
  },
  {
    name: 'Terminator',
    year: '1983',
    review: `Once again, Schwarzenegger shows why he's the boss.`,
    privateComments: `Terminator is my favorite movie ever. `,
  },
  {
    name: 'Jaws',
    year: '1971',
    review: 'The original blockbuster.',
    privateComments: `I'm scared of sharks…`,
  },
  {
    name: 'Die Hard II',
    year: '1991',
    review: `Another classic.`,
  },
  {
    name: 'Rush Hour',
    year: '1993',
    review: `Jackie Chan at his best.`,
  },
  {
    name: 'Citizen Kane',
    year: '1943',
    review: `A disappointing lack of action sequences.`,
  },
  {
    name: 'Commando',
    year: '1983',
    review: 'A good contender for highest kill count ever.',
  },
];

Meteor.startup(function () {
  if (Users.find().fetch().length === 0) {
    Accounts.createUser({
      username: 'DemoUser',
      email: 'dummyuser@telescopeapp.org',
      profile: {
        isDummy: true
      }
    });
  }
  const currentUser = Users.findOne();
  if (Movies.find().fetch().length === 0) {
    seedData.forEach(document => {
      newMutation({
        action: 'movies.new',
        collection: Movies,
        document: document, 
        currentUser: currentUser,
        validate: false
      });
    });
  }
});
```

This time, we don't want to import this on the client, so we'll import it directly from our `server/main.js` endpoint:

```js
import '../modules/index.js';
import './seed.js';
```

Head to [http://localhost:3000/graphiql](http://localhost:3000/graphiql) and type:

```js
query moviesQuery{
  moviesList{
    createdAt
    name
  }
}
```

You should get a list of movie names and creation dates on the right. If you'd like, try requesting more fields:

```js
query moviesQuery{
  moviesList{
    createdAt
    name
    review
    year
  }
}
```

As you can see, the great thing about GraphQL is that you can specify exactly which piece of data you need!

## More Advanced Resolvers*

*(Note: you can skip this section if you're using default resolvers and mutations)*

Although our resolver works, it's fairly limited. As it stands, we can't filter, sort, or paginate our data. Even though we might not need these features right away, now is a good time to set things up in a more future-proof way.

Our resolver will accept a [terms](terms-parameters.html) object that can specify filtering and sorting options, which is then transformed into a MongoDB-compatible object by the `Movies.getParameters()` function. Additionally, we'll add a second resolver that takes in the same `terms`, but returns the *number* of documents matching these terms:

```js
const resolvers = {

  list: {

    name: 'moviesList',

    resolver(root, {terms = {}}, context, info) {
      const {selector, options} = context.Movies.getParameters(terms, {}, context.currentUser);
      return context.Movies.find(selector, options).fetch();
    },

  },

  total: {
    
    name: 'moviesTotal',
    
    resolver(root, {terms = {}}, context) {
      const {selector, options} = context.Movies.getParameters(terms, {}, context.currentUser);
      return context.Movies.find(selector, options).count();
    },
  
  },
};

export default resolvers;
```

Our resolvers take three arguments:

- `root`: a link back to the root of the current GraphQL node. You can safely ignore this for now. 
- `args`: an object containing the query arguments passed from the client.
- `context`: an object enabling access to our collections, as well as utilities such as `getViewableFields` (which is used to get a list of all fields viewable for the current user for a given collection). 

Once we've figured out the correct `selector` and `options` object from the `terms` and the current user, all that's left is to make the actual database query using `Movies.find()`. 

## The User Resolver

If you inspect the collection schema for a movie, you'll notice it contains a `userId` field that contains a string. 

But that string, while useful internally, isn't much use to our app's users. Instead, we'd much rather display the reviewer's name. In order to do so, we'll **resolver** the `userId` field as a `user` object containing all the data we need. 

Go back to your `schema.js` file, and add the following `resolveAs` property to the `userId` field:

```js
userId: {
  type: String,
  optional: true,
  viewableBy: ['guests'],
  resolveAs: {
    fieldName: 'user',
    type: 'User',
    resolver: (movie, args, context) => {
      return context.Users.findOne({ _id: movie.userId }, { fields: context.Users.getViewableFields(context.currentUser, context.Users) });
    },
    addOriginalField: true
  }
},
```

We are doing four things here:

1. Specifying that the field should be named `user` in the API.
2. Specifying that the `user` field returns an object of GraphQL type `User`.
3. Defining a `resolver` function that indicates how to retrieve that object.
4. Specifying that in addition to this new `user` object, we want the original `userId` field to still be available as well. 

## Displaying Data

Now that we know we can access data from the client, let's see how to actually load and display it within our app. 

We'll need two pieces for this: a [**container** component](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.uid3w3pk8) that loads the data, and a **presentational** component that displays it. Fortunately, Vulcan comes with [a set of built-in higher-order container components](data-loading.html) which we can use out of the box, so we can focus on the presentational components.

Create a new `MoviesItem` component inside `components`:

```js
import React, { PropTypes, Component } from 'react';
import { registerComponent, ModalTrigger } from 'meteor/vulcan:core';

const MoviesItem = ({movie, currentUser}) =>

  <div style={ { paddingBottom: "15px",marginBottom: "15px", borderBottom: "1px solid #ccc" } }>

    <h4>{movie.name} ({movie.year})</h4>
    <p>{movie.review} – {movie.user && movie.user.displayName}</p>

  </div>

export default MoviesItem;
```

We'll also come back to the `MoviesList` component and use a [GraphQL fragment](fragments.html) (which we'll define in the next section) to specify what data we want to load using the `withList` [higher-order component](https://facebook.github.io/react/docs/higher-order-components.html): 

```js
import React, { PropTypes, Component } from 'react';
import { Components, withList, withCurrentUser, Loading } from 'meteor/vulcan:core';

import Movies from '../../modules/movies/collection.js';
import MoviesItem from './MoviesItem.jsx';

const MoviesList = ({results = [], currentUser, loading, loadMore, count, totalCount}) => 
  
  <div style={ { maxWidth: '500px', margin: '20px auto' } }>

    {/* user accounts placeholder*/}

    {loading ? 

      <Loading /> :

      <div className="movies">
        
        {/* new document form placeholder */}

        {/* documents list */}

        {results.map(movie => <MoviesItem key={movie._id} movie={movie} currentUser={currentUser} />)}
        
        {/* load more */}

        {totalCount > results.length ?
          <a href="#" onClick={e => {e.preventDefault(); loadMore();}}>Load More ({count}/{totalCount})</a> : 
          <p>No more items.</p>
        }

      </div>
    }

  </div>

const options = {
  collection: Movies,
  fragmentName: 'MoviesItemFragment',
  limit: 5
};

export default withList(options)(withCurrentUser(MoviesList))
```

We want to provide `MoviesList` with the `results` document list and the `currentUser` property, so we wrap it with `withList` and `withCurrentUser`.

## Fragments

At this stage, we'll probably get an error message saying the fragment we're using is not yet defined. To remedy this, create a new `fragments.js` file inside `modules/movies`:

```js
import { registerFragment } from 'meteor/vulcan:core';

registerFragment(`
  fragment MoviesItemFragment on Movie {
    _id
    createdAt
    userId
    user {
      displayName
    }
    name
    year
    review
  }
`);
```

And don't forget to import it within `collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import './fragments.js';

//...
```

Once you save, you should finally get your prize: a shiny new movie list displayed right there on your screen!

## User Accounts

So far so good, but we can't yet do a lot with our app. In order to give it a little more potential, let's add user accounts to `MoviesList`: 

```js
import React, { PropTypes, Component } from 'react';
import { Components, withList, withCurrentUser, Loading } from 'meteor/vulcan:core';

import Movies from '../../modules/movies/collection.js';
import MoviesItem from './MoviesItem.jsx';

const MoviesList = ({results = [], currentUser, loading, loadMore, count, totalCount}) => 
  
  <div style={ { maxWidth: '500px', margin: '20px auto' } }>

    {/* user accounts */}

    <div style={ { padding: '20px 0', marginBottom: '20px', borderBottom: '1px solid #ccc' } }>
    
      <Components.AccountsLoginForm />
    
    </div>

    {loading ? 

      <Loading /> :

      <div className="movies">
        
        {/* new document form placeholder */}

        {/* documents list */}

        {results.map(movie => <MoviesItem key={movie._id} movie={movie} currentUser={currentUser} />)}
        
        {/* load more */}

        {totalCount > results.length ?
          <a href="#" onClick={e => {e.preventDefault(); loadMore();}}>Load More ({count}/{totalCount})</a> : 
          <p>No more items.</p>
        }

      </div>
    }

  </div>

const options = {
  collection: Movies,
  fragmentName: 'MoviesItemFragment',
  limit: 5
};

export default withList(options)(withCurrentUser(MoviesList));
```

Yay! You can now log in and sign up at your own leisure. Note that the `<Components.AccountsLoginForm />` component is a ready-made accounts UI component that comes from the `vulcan:accounts` package. 

## Mutations*

*(Note: you can skip this section if you're using default resolvers and mutations)*

Now that we're logged in, we can start interacting with our data. Let's build a simple form for adding new movies.

Before we can build the user-facing part of this feature though, we need to think about how the insertion will be handled server-side, using [Mutations](mutations.html).

Create a new `mutations.js` file in `modules/movies`:

```js
import { newMutation, Utils } from 'meteor/vulcan:core';
import Users from 'meteor/vulcan:users';

const mutations = {

  new: {
    
    name: 'moviesNew',
    
    check(user) {
      if (!user) return false;
      return Users.canDo(user, 'movies.new');
    },
    
    mutation(root, {document}, context) {
      
      Utils.performCheck(this.check, context.currentUser, document);

      return newMutation({
        collection: context.Movies,
        document: document, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  }

};

export default mutations;
```

This mutation performs a simple check for the presence of a logged-in user and whether they can perform the action, and then passes on the `document` property to one of Vulcan's boilerplate mutations, `newMutation`. 

Let's pass it on to our `createCollection` function in `collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import './fragments.js';
import mutations from './mutations.js';

const Movies = createCollection({

  collectionName: 'Movies',

  typeName: 'Movie',

  schema,
  
  resolvers,

  mutations,

});

export default Movies;
```

## Actions, Groups, & Permissions

The mutation's `check` function checks if the user can perform an action named `movies.new`. We want all logged-in users (known as the `members` group) to be able to perform this action, so let's take care of this by creating a new `movies/permissions.js` file:

```js
import Users from 'meteor/vulcan:users';

const membersActions = [
  'movies.new',
];
Users.groups.members.can(membersActions);
```

And adding it to `collection.js` as usual:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import './fragments.js';
import mutations from './mutations.js';
import './permissions.js';

//...
```

Note that in this specific case, creating an action and checking for it is a bit superfluous, as it boils down to checking if the user is logged in. But this is a good introduction to the permission patterns used in Vulcan, which you can learn more about in the [Groups & Permissions](/groups-permissions.html) section.

One more thing! By default, all schema fields are locked down, so we need to specify which ones the user should be able to insert as part of a “new document” operation. 

Once again, we do this through the schema. We'll add an `insertableBy` property to any “insertable” field and set it to `[members]` to indicate that a field should be insertable by any member of the `members` group (in other words, regular logged-in users):

```js
const schema = {

  // default properties

  _id: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
  },
  createdAt: {
    type: Date,
    optional: true,
    viewableBy: ['guests'],
    onInsert: (document, currentUser) => {
      return new Date();
    }
  },
  userId: {
    type: String,
    optional: true,
    viewableBy: ['guests'],
    resolveAs: {
      fieldName: 'user',
      type: 'User',
      resolver: (movie, args, context) => {
        return context.Users.findOne({ _id: movie.userId }, { fields: context.Users.getViewableFields(context.currentUser, context.Users) });
      }
    }
  },
  
  // custom properties

  name: {
    label: 'Name',
    type: String,
    optional: true,
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members'],
  },
  year: {
    label: 'Year',
    type: String,
    optional: true,
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members'],
  },
  review: {
    label: 'Review',
    type: String,
    optional: true,
    control: 'textarea',
    viewableBy: ['guests'],
    insertableBy: ['members'],
    editableBy: ['members']
  },

};

export default schema;
```

While we're at it we'll also specify which fields can be edited via `editableBy`. In this case, the three fields we want members to be able to write to are `name`, `year`, and `review`. Finally, we'll also give the `review` field a `textarea` form control. 

At this point it's worth pointing out that for mutations like inserting and editing a document, we have two distinct permission "checkpoints": first, the mutation's `check` function checks if the user can perform the mutation at all. Then, each of the mutated document's fields is checked individually to see if the user should be able to mutate it.

This makes it easy to set up collections with admin-only fields, for example. 

## Forms

We now have everything we need to create a new `MoviesNewForm.jsx` component using the [SmartForm](/forms.html) utility:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent, withCurrentUser, getFragment } from 'meteor/vulcan:core';

import Movies from '../../modules/movies/collection.js';

const MoviesNewForm = ({currentUser}) =>

  <div>

    {Movies.options.mutations.new.check(currentUser) ?
      <div style={ { marginBottom: '20px', paddingBottom: '20px', borderBottom: '1px solid #ccc' } }>
        <h4>Insert New Document</h4>
        <Components.SmartForm 
          collection={Movies}
          mutationFragment={getFragment('MoviesItemFragment')}
        /> 
      </div> :
      null
    }

  </div>

export default withCurrentUser(MoviesNewForm);
```

A few things to note: 

- We'll pass the `MoviesItemFragment` fragment to the form so that it knows what data to return from the server once the mutation is complete. 
- We only want to show the “New Movie” form when a user actually *can* submit a new movie, so we'll make use of the `new` mutation's `check` function to figure this out.
- We need to access the current user to perform this check, so we'll use the `withCurrentUser` higher-order component. 

Let's import the form component from `MoviesList.jsx`:

```js
import React, { PropTypes, Component } from 'react';
import { Components, withList, withCurrentUser, Loading } from 'meteor/vulcan:core';

import Movies from '../../modules/movies/collection.js';
import MoviesItem from './MoviesItem.jsx';
import MoviesNewForm from './MoviesNewForm.jsx';

const MoviesList = ({results = [], currentUser, loading, loadMore, count, totalCount}) => 
  
  <div style={ { maxWidth: '500px', margin: '20px auto' } }>

    {/* user accounts */}

    <div style={ { padding: '20px 0', marginBottom: '20px', borderBottom: '1px solid #ccc' } }>
    
      <Components.AccountsLoginForm />
    
    </div>

    {loading ? 

      <Loading /> :

      <div className="movies">
        
        {/* new document form */}

        <MoviesNewForm />

        {/* documents list */}

        {results.map(movie => <MoviesItem key={movie._id} movie={movie} currentUser={currentUser} />)}
        
        {/* load more */}

        {totalCount > results.length ?
          <a href="#" onClick={e => {e.preventDefault(); loadMore();}}>Load More ({count}/{totalCount})</a> : 
          <p>No more items.</p>
        }

      </div>
    }

  </div>

const options = {
  collection: Movies,
  fragmentName: 'MoviesItemFragment',
  limit: 5
};

export default withList(options)(withCurrentUser(MoviesList));
```

Now fill out the form and submit it. The query will be updated and the new movie will appear right there in our list! 

## Editing Movies

We're almost done! All we need now is to add a way to edit movies. First, create a new `MoviesEditForm.jsx` component:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent, getFragment } from "meteor/vulcan:core";

import Movies from '../../modules/movies/collection.js';

const MoviesEditForm = ({documentId, closeModal}) =>

  <Components.SmartForm 
    collection={Movies}
    documentId={documentId}
    mutationFragment={getFragment('MoviesItemFragment')}
    showRemove={true}
    successCallback={document => {
      closeModal();
    }}
  />

export default MoviesEditForm;
```

Because we're passing a specific `documentId` property to the `SmartForm` component, this form will be an **edit document** form. We're also passing `showRemove` to show a "delete document" option, and a `successCallback` function to close the modal popup inside which the form will be displayed. 

But note that to *edit* a document, we first need to know what that document *is*. In other words, we need to load the document on the client. SmartForms can take care of this for us, but we do need to write a new resolver. After all, the one we have displays a *list* of document, not a *single* document. 

Go back to `resolvers.js` and add a `single` resolver:

```js
const resolvers = {

  list: {

    name: 'moviesList',

    resolver(root, {terms = {}}, context, info) {
      const {selector, options} = context.Movies.getParameters(terms, {}, context.currentUser);
      return context.Movies.find(selector, options).fetch();
    },

  },

  single: {

    name: 'moviesSingle',

    resolver(root, {documentId}, context) {
      return context.Movies.findOne(documentId);
    }
  },

  total: {
    
    name: 'moviesTotal',
    
    resolver(root, {terms = {}}, context) {
      const {selector, options} = context.Movies.getParameters(terms, {}, context.currentUser);
      return context.Movies.find(selector, options).count();
    },
  
  },
};

export default resolvers;
```

Now let's go back to `MoviesItem` and make use of our form:

```js
import React, { PropTypes, Component } from 'react';
import { Components, registerComponent } from 'meteor/vulcan:core';

import Movies from '../../modules/movies/collection.js';
import MoviesEditForm from './MoviesEditForm.jsx';

const MoviesItem = ({movie, currentUser}) =>

  <div style={ { paddingBottom: "15px",marginBottom: "15px", borderBottom: "1px solid #ccc" } }>

    {/* document properties */}
    
    <h4>{movie.name} ({movie.year})</h4>
    <p>{movie.review} – {movie.user && movie.user.displayName}</p>
    
    {/* edit document form */}

    {Movies.options.mutations.edit.check(currentUser, movie) ? 
      <Components.ModalTrigger label="Edit Movie">
        <MoviesEditForm currentUser={currentUser} documentId={movie._id} />
      </Components.ModalTrigger>
      : null
    }

  </div>

export default MoviesItem;
```

This time we're using the `<Components.ModalTrigger />` Vulcan component to show our form inside a popup (assuming the current user can perform an edit, of course).

Finally, we'll also need to hook up our edit and remove mutations in `mutations.js`:

```js
import { newMutation, editMutation, removeMutation, Utils } from 'meteor/vulcan:core';
import Users from 'meteor/vulcan:users';

const mutations = {

  new: {
    
    name: 'moviesNew',
    
    check(user) {
      if (!user) return false;
      return Users.canDo(user, 'movies.new');
    },
    
    mutation(root, {document}, context) {
      
      Utils.performCheck(this.check, context.currentUser, document);

      return newMutation({
        collection: context.Movies,
        document: document, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  },

  edit: {
    
    name: 'moviesEdit',
    
    check(user, document) {
      if (!user || !document) return false;
      return Users.owns(user, document) ? Users.canDo(user, 'movies.edit.own') : Users.canDo(user, `movies.edit.all`);
    },

    mutation(root, {documentId, set, unset}, context) {

      const document = context.Movies.findOne(documentId);
      Utils.performCheck(this.check, context.currentUser, document);

      return editMutation({
        collection: context.Movies, 
        documentId: documentId, 
        set: set, 
        unset: unset, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  },
  
  remove: {

    name: 'moviesRemove',
    
    check(user, document) {
      if (!user || !document) return false;
      return Users.owns(user, document) ? Users.canDo(user, 'movies.remove.own') : Users.canDo(user, `movies.remove.all`);
    },
    
    mutation(root, {documentId}, context) {

      const document = context.Movies.findOne(documentId);
      Utils.performCheck(this.check, context.currentUser, document);

      return removeMutation({
        collection: context.Movies, 
        documentId: documentId, 
        currentUser: context.currentUser,
        validate: true,
        context,
      });
    },

  },

};

export default mutations;
```
For the `check` function to work properly we need to update the `permissions.js` file.

```js
import Users from 'meteor/vulcan:users';

const membersActions = [
  'movies.new',
  'movies.edit.own',
  'movies.remove.own',
];
Users.groups.members.can(membersActions);
```

All done! You should now be able to edit and remove movies. 

## Sorting

As it stands, our movie list isn't really sorted. What if we wanted to sort it by movie year? In Vulcan, document lists receive their options via a `terms` object. And whenever that object is processed (whether on the client or server), it goes through a serie of successive callbacks. So if we want to set a default sort, we can do so through one of these callbacks. 

Create a `parameters.js` file (learn more about [terms and parameters here](terms-parameters.html)):

```js
import { addCallback } from 'meteor/vulcan:core';

function sortByYear (parameters, terms) {
  return {
    selector: parameters.selector, 
    options: {...parameters.options, sort: {year: -1}}
  };
}

addCallback("movies.parameters", sortByYear);
```

And add it to `movies/collection.js`:

```js
import { createCollection } from 'meteor/vulcan:core';
import schema from './schema.js';
import resolvers from './resolvers.js';
import './fragments.js';
import mutations from './mutations.js';
import './permissions.js';
import './parameters.js';

//...
```

Note that the list automatically re-sorts as you edit individual documents. This might seem like a simple feature, but it's one more thing you'd have to implement yourself if you weren't using Vulcan!

## Going Further

This is probably a good place to stop, but you can go further simply by going through the code of the `example-instagram` package. In it, you'll see how to create a resolver for single documents so you can load more data for a specific movie, and use permission checks to enforce fine-grained security throughout your app.

And this is just the start. You can do a lot more with Vulcan, Apollo, and React, as you'll soon see!
