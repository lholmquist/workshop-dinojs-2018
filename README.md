# DinosaurJS 2018 Serverless Workshop

Welcome to the Serverless Design workshop at DinosaurJS 2018!

In this workshop, we'll be using [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) to create a simple but functional serverless application. We have Azure credits to give everyone, if you did not receive an Azure pass, please request one from the Bryan Hughes, workshop instructor.

## Part 1: Hello World

For this first section, Bryan will lead you step by step through creating a hello world example in Azure Functions. We'll create a function that takes in a name in an HTTP query, and returns a JSON response saying hi.

## Part 2: Adding a Database

For this section, Bryan will lead you step by step through adding a [Cosmos DB](https://azure.microsoft.com/en-us/services/cosmos-db/) database to store a list of dinosaurs, and return the list to a user in JSON.

## Part 3: Moar Functions!

For this self-guided section, you will modify your first function and implement two new function to serve as the back-end for the front-end implemented at [index.html](index.html). You can load this front-end directly from your filesystem, there is no need to use an HTTP server to host it. Just make sure to set CORS to `*` in the Azure Portal!

This frontend allows you to view all dinosaurs in the list, including a photo of the dinosaur, and to add new ones. Specifically, you will need to implement two HTTP endpoints that the front-end will call.

## PUT /api/items Function

The first endpoint is completely new, and will allow us to create items and store them in Cosmos DB.

The request body should have the following structure:

```
PUT /api/items
{
  name: string;
  type: string;
  imageURL: string;
}
```

Note that we have added a new field here called `imageURL`. This field allows users to specify a URL to an image (e.g. https://upload.wikimedia.org/wikipedia/commons/f/fb/Stegosaurus_stenops_sophie_wiki_martyniuk.png). This url should be stored in the database along with the rest of the dinosaur's information.

### GET /api/items Function

The next endpoint is a slight modification of the `/api/hello` endpoint function we created in the previous section.

The response body should have the following structure:

```
GET /api/items
[{
  id: string;
  name: string;
  type: string;
  image: string; // Base 64 encoded image data URI
}, ... ]
```

Most of this is the same as before, but note how we have added a new field called `image` that contains a [Base 64 encoded data URI](https://css-tricks.com/data-uris/) version of the image found at the URL specified above for the dinosaur.

There is an extra requirement for this excercise involving images: you must fetch it from another Azure Function. You should use the [request](https://www.npmjs.com/package/request) module from npm to make an HTTP GET request to the Azure Function at `GET /api/image/?url=${imageURL}`.

For bonus points, add a cache to this Azure Function so that you only have to call the other Azure Function once per url. You can do this using a JavaScript [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) or a plain old JavaScript object. You should use the image URL as the key.

## Image Fetch Function

This Azure Function is the image helper function that was mentioned above, and will exist at `GET /api/image/?url=${imageURL}`. The return value will be the data URI encoded version of the image. Take care that you get the correct header for the correct file type in the data URI, e.g. `data:image/png;base64,` for a PNG.

For bonus points, add a cache to this Azure Function that behaves like the cache in the other Azure Function. You might be asking yourself "why would I cache in both places?" In a normal monolith application, there wouldn't be a need, but it can be needed here! If, for example, the GET function that calls this one goes cold while this one is hot, the GET function's cache would be empty, but this one wouldn't be.

## Stretch Goals

### Bundling

Once you've implemented your Azure Functions and gotten everything working correctly, let's improve cold-start time by using [Azure Function Pack](https://github.com/Azure/azure-functions-pack) to bundle your source and dependencies into a single file.

### Debugging

Console debugging can only get us so far, and Azure Functions has a number of tools for using a debugger in Visual Studio and Visual Studio Code.

For Visual Studio Code, insteall the Azure extension and follow the instructions at https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions to set up debugging for Azure Functions

# License

MIT License

Copyright (c) 2018 Bryan Hughes <bryan.hughes@microsoft.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

