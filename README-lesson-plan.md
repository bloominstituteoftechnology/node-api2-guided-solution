# Web API II Guided Project

Guided project for **Web API II** module.

In this project we will learn how to create a very simple Web API using `Node.js` and `Express`, and cover how to use `Express Routers` to break up the application to make it more maintainable.

## Prerequisites

- a REST client like [insomnia](https://insomnia.rest/download/) or [Postman](https://www.getpostman.com/downloads/) installed.

## Starter Code

The [Starter Code](https://github.com/BloomInstituteOfTechnology/node-api2-guided) for this project is configured to run the server by typing `npm run server`. The server will restart automatically on changes.

Data for the API will be stored in a database and will persist across server restarts.

## Introduce REST

Load Canvas and walk students through a **brief** introduction to `REST`. Emphasis on it being a set of recommendations, a way of thinking about the architecture of distributed systems.

Quickly explain the 6 constraints.

Use this table to contrast RESTful endpoints from non RESTful counterparts.

| Not using REST      | Using REST                |
| ------------------- | --------------------------|
| `/listAllAdopters`  | GET    /adopters          |
| `/createAdopter`    | POST   /adopters          |
| `/updateAdopter`    | PUT    /adopters/:id      |
| `/deleteAdopter`    | DELETE /adopters/:id      |
| `/listAdopterDogs`  | GET    /adopters/:id/dogs |

Next we'll start breaking up the monolithic `index.js` file into separate files to achieve **separation of concerns** and make the code easier to maintain in the future.

We start by separating the creation and configuration of our server from starting to listen for requests.

## Preliminaries

- make sure to download all dependencies with `npm i`.
- use `npm run server` to run the API and visit `/api/adopters` to make sure it's working.
- explain the structure of the project showing the monolithic `server.js` file.
- explain the `CommonJS` way of exporting modules and its differences with ES6 modules.
- show the functions inside `adopters/adopters-model.js` and explain how they use a real SQLite database.

Next, we'll break the application into sub-applications using Express Routers.

## Add Adopters Router

- inside the `api/adopters` folder add a new file, call it `adopters-router.js`.
- add this content to `adopters-router.js`. Note how similar it is to creating a full server.

```js
const express = require('express');

const router = express.Router();
// Take time to explain that a Router can have routes and middleware just like a full server can.

module.exports = router;
```

- move all endpoints for `/api/adopters` from `server.js` to the newly created router.
- move the `require` for the `adopters-model` from server to the router.
- refactor all instances of `server.` to read `router.` as those endpoints now belong to the router and not the server.
- remove `/api/adopters` from the URL of all the endpoints. Explain that this router only cares about what comes after `/api/adopters`. **Take your time to explain this step**.

After all the changes, `adopters-router.js` file looks like this:

```js
const express = require('express');

// models commonly use the singular
const Adopter = require('./adopters-model.js');

const router = express.Router();

// handles GET /api/adopters/
router.get('/', async (req, res) => {
  // same implementation as before omitted to save space
});

// handles GET /api/adopters/:id
router.get('/:id', async (req, res) => {
  // same implementation as before omitted to save space
});

// handles POST /api/adopters/
router.post('/', async (req, res) => {
  // same implementation as before omitted to save space
});

// etc

module.exports = router;
```

- the last step is to go back to `api/server.js` and `use` the router for all routes that begin with `/api/adopters`.

```js
const adoptersRouter = require('./adopters/adopters-router'); // add this line after requiring express

server.use(express.json());
server.use('/api/adopters', adoptersRouter); // add this line to use the router
```

**wait for students to catch up, use a `yes/no` poll to let students tell you when they are done**

## Reading Query String Parameters

We have covered how to receive data from the client using the request `body` and `url parameters`. Now we'll explore another way to accept data from the client.

1. Create a file markdown file (notes.md is what I use) for the next step.
2. Do a quick search in google for `query string parameters`.
3. Copy the URL from the browser and paste it in your markdown file. It looked like this for me:

```md
https://www.google.com/search?newwindow=1&source=hp&ei=Z1drXP_JBYS0ggepyrmIAQ&q=query+string+parameters&btnK=Google+Search&oq=query+string+parameters&gs_l=psy-ab.3..0l10.1617.6689..6917...2.0..0.122.2798.2j25......0....1..gws-wiz.....0..0i131j0i10._0l3Msv44yw
```

4. Use the URL to introduce the concept of `query string parameters`.
5. break it up showing how query string parameters work:

```txt
https://www.google.com/search
?           <--marks the beginning of the query string
newwindow=1 <-- a key value/pair
&           <-- separator for the next key/value pair
source=hp   <-- rinse and repeat
&
ei=Z1drXP_JBYS0ggepyrmIAQ
delete the rest
```

Explain that `Express` will take the query string, parse it and save it as an object in the request under `req.query`. In that example the resulting `req.query` would look like this:

```js
req.query = {
  newwindow: '1',
  source: 'hp',
  ei: 'Z1drXP_JBYS0ggepyrmIAQ',
};
```

Let's see this in action. Notice this endpoint:

```js
router.get('/', (req, res) => {
  Adopter.find(req.query)
  .then(adopter => {
    res.status(200).json(adopter);
  })
  .catch(error => {
    console.log(error);
    res.status(500).json({
      message: 'Error retrieving the adopter',
    });
  });
});
```

Now let's look at the `.find()` method of `adopters-model.js`. It is set to read `page`, `limit`, `sortby` and `sortdir` from the query string and use it to do pagination and sorting of the records at the database level.

```js
function find(query) {
  const { page = 1, limit = 2, sortby = 'id', sortdir = 'asc' } = query;
  const offset = limit * (page - 1);

  const rows = db('adopters')
    .orderBy(sortby, sortdir)
    .limit(limit)
    .offset(offset);

  return rows;
}
```

Students will learn about how that works in the database week, for now we'll cover how to send parameters throw the query string.

- Use `Postman` to make a request to `/api/adopters`, notice we only see 2 records.
- change the url to include the `limit` query string parameter: `/api/adopters?limit=3`. This makes the page size be 3 records.
- change the url to include the `page` query string parameter: `/api/adopters?limit=3&page=2`. Now the next 2 records are shown.
- change the url to include the `sortby` and `sortdir` query string parameters: `/api/adopters?limit=3&page=2&sortby=name&sortdir=desc`.

**wait for students to catch up, use a `yes/no` poll to let students tell you when they are done**

Next, let's see how to handles sub-resources by building an endpoint to view all dogs for a rescuer with a given `id`.

## Use sub-routes

When a resource makes sense within the context of another resource, in REST we can create a sub-route. For our use case, we are interested in the dogs adopted by a particular adopter:

```js
router.get('/:id/dogs', (req, res) => {
  Adopter.findDogs(req.params.id)
    .then(dogs => {
      if (dogs.length > 0) {
        res.status(200).json(dogs);
      } else {
        res.status(404).json({ message: 'No dogs for this adopter' });
      }
    })
    .catch(error => {
      console.log(error);
      res.status(500).json({
        message: 'Error retrieving the dogs for this adopter',
      });
    });
});

```

## Gotcha, show if time permits

Sometimes the following error displays in the console `Cannot set header after they are sent to the client`.

This code suffers from that:

```js
router.get('/:id', (req, res) => {
  Adopter.findById(req.params.id)
    .then(adopter => {
      if (adopter) {
        res.status(200).json(adopter);
      }
      // we forget the else block!!
      // this code tries to modify the response a second time, this is the reason for the error
      res.status(404).json({ message: 'Adopter not found' });
    })
    .catch(error => {
      console.log(error);
      res.status(500).json({
        message: 'Error retrieving the adopter',
      });
    });
});
```

- fix it by wrapping the `404` in an `else` or changing the success line to `return res.status(200).json(hub);`

```js
if (adopter) {
  res.status(200).json(adopter);
} else {
  res.status(404).json({ message: 'Adopter not found' });
}
```
