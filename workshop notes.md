# Workshop Notes

## Part 1: Hello World

We'll create a working Hello World example, and make it a bit more JavaScript-y

1. Create Resource -> Compute -> Function App
    1. Name it "nebrius-dinojs-2018"
    1. OS: Windows (mention Docker is typically overkill given purpose of functions, and OS isn't really apparent in most cases)
    1. Application Insights: off (but mention what it is)
    1. Pin to dashboard
1. nebrius-dinojs-2018 -> Functions -> +
    1. Webhook+API
    1. JavaScript
1. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1
    1. "Get function URL"
    1. Past in new tab in Browser. Mention it's XML, which we don't like
    1. Change context.res in code to include:

```JavaScript
      headers: {
        'Content-Type': 'application/json'
      }
```

4. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1 -> Integrate
    1. Change "Authorization Level" to "anonymous"
    1. Change the route to "hello"
    1. Show off difference in "Get URL" in main function page

Make note of `context.log` vs `console.log`

## Part 1.1: Anatomy of a Function

Just a quick tour around everything here, no major changes.

1. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1 -> Integrate
    1. Show off allowed HTTP methods
    1. Advanced editor, show off function.json file. Mention this must be done by hand for auto-deploys from GitHub, great to grab from here
1. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1 -> Manage
    1. Adding environment variables
    1. Monitor, mention what App Insights is
1. nebrius-dinojs-2018 -> Overview -> Function App Settings
    1. Daily Usage Quota
    1. Read-only mode
1. nebrius-dinojs-2018 -> Overview -> Application Settings
    1. Talk briefly about them all
    1. Change the Node.js version to `8.9.4`
1. nebrius-dinojs-2018 -> Overview -> Platform Features
    1. Show off Kudu, and show off code structure (function.json, etc)
    1. Show off deployment, and how to connect to GitHub

## Part 2: Adding CosmosDB

We'll modify the existing to get data from the DB

1. Create Resource -> Databases -> Cosmos DB
    1. Name it "nebrius-dinojs-2018-db"
    1. API: SQL (Note that Mongo support isn't there yet)
    1. Resource Group: Use existing
    1. Pin to Dashboard
1. nebrius-dinojs-2018-db -> Quick start (should be there by default) -> Node.js
    1. Create items collection
1. nebrius-dinojs-2018-db -> Data Explorer
    1. Show off what's already there
    1. Create new document with

```JSON
{
    "id": "1",
    "type": "stegosaurus",
    "name": "Alice"
}
```

4. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1 -> Integrate
    1. New Input, select Cosmos DB
    1. Document Parameter Name: "dinosaurs"
    1. Database Name: "ToDoList"
    1. Collection Name: "Items"
    1. Azure Cosmos DB account connection: click "new"
1. Rewrite the code to be:

```JavaScript
module.exports = function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');
    context.res = {
        // status: 200, /* Defaults to 200 */
        body: context.bindings.dinosaurs,
        headers: {
            'Content-Type': 'application/json'
        }
    };
    context.done();
};
```

6. Run again, and see data from the database

## Part 2.1: Dependencies

We'll add the request package

1. `npm init` in workshop dir
    1. Add `request` to `dependencies` at version `^2.87.0`
2. nebrius-dinojs-2018 -> Functions -> HttpTriggerJS1 -> Overview -> View Files -> Upload
    1. Upload this package.json
    2. Open Kudo and run `npm install`
        1. Mention this is only required when not doing CI/CD
    3. Add `const request = require('request')` to file
    4. Run it using portal debug tools to show that it doesn't crash, but do nothing else

## Part 2.2: Design philosophies

This is a section to just talk to the audience

1. Serverless is the next evolution of microservices. Serverless to microservices is similar to microsoervices to monoliths
1. Embrace automation and the "serverless way." It may be new, but it's more effecient
1. Serverless is like functional programming, ideally stateless. Think of it as intelligently wiring inputs to outputs.
1. UNIX way: small tools that do one thing well

## Part 2.3: Optimizing

Start by talking about more practical considerations, before getting back to code.

1. Startup time is crucial, focus on getting it down
    1. There is what's called cold starts and hot starts...same as VMs and PaaS have.
    1. Hot starts are fast, cause the Node.js process is already running
    2. Cold starts are expensive. They must spin up a container/etc and run the apps startup code. Also file system access is very slow with lots of deps
    3. Can't do much about spinning up a container, although it sometimes is possible to create a "keep-alive" ping
    4. Can remove dependencies (e.g. using input/output bindings instead of your own code to connect to a DB)
    5. Do as much async initialization as possible, try and remove what you can
2. You can do some caching in the function
    1. Create a global variable to store things like DB results, etc
    2. If your function remains hot, that cache will stay there
    3. Always make sure to check if thing is cached before using cache

## Part 3: Self-directed

Remember to update CORS

PUT /api/items {
  name: string;
  type: string;
  imageUrl: string;
}

GET /api/items {
  id: string;
  name: string;
  image: string; // Base 64 encoded image
}
