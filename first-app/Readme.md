# pNodeJs

### Base code with comments
```js
// Dependencies
const http = require('http');
// The url library gives us all the methods related to url
const url = require('url');
// stringDecoder to get the payload
const StringDecoder = require('string_decoder').StringDecoder;

// The server should respond to all requests with a string
const server = http.createServer((req, res) => {
  // Here we get the requested url and parse it, we use true to get the querystrings along with the url, this gives us other methods on parsedUr;
  const parsedUrl = url.parse(req.url, true);
 // This will give us the pathname that we are looking for ...eg .. localhost:3000/bar ---> bar (is the pathname) 
  const path = parsedUrl.pathname;
  // trimmed pathname trims the extra / slashes on both sides ...eg ..localhost:3000/foo/ ==> foo 
  const trimmedPathName = path.replace(/^\/+|\/+$/g, '');
  // Get the querystrings from the url
  const queryStringObject = parsedUrl.query;
  // To get the method used
  const method = req.method;
  // To get the headers, set them usng postman
  const headers = req.headers;
  // get he payload
  // payloads come in a http server as a stream. Hence we need to save it in a buffer variable.
  // Each time we get a stream of data, this req.on('data') runs 
  const decoder = new StringDecoder('utf-8'); // we will get the utf-8 format of payload
  let buffer = '';
  req.on('data', (data) => {
    // append inside buffer, the decoded steam of data into utf-8
    buffer += decoder.write(data);
  });
  // when the request ends, the stream ends, so we use the end method to write this
  // this end event is always going to be called irrespective of the presence of payload
  req.on('end', () => {
    buffer += decoder.end(); // end the appending to buffer variable
    console.log('request recieved on path', trimmedPathName);
    console.log('querystrings', queryStringObject);
    console.log('headers: ', headers);
    console.log('payload', buffer);
    console.log('method', method);
    // choose the handler this request should go to, if one is not found, redirect to 404
    const chosenHandler = typeof(router[trimmedPathName]) !== 'undefined' ? router[trimmedPathName] : handlers.notFound;
    // Construct the data
    const data = {
      trimmedPathName,
      queryStringObject,
      headers,
      payload: buffer,
      method
    }; 
    console.log('chosenHandler', typeof(chosenHandler), chosenHandler);
    // Route the request to the chosen handler specified in the router.
    chosenHandler(data, (statusCode, payload) => {
      // Define some defaults cos some cb may have code and no payload like 404 etc.
      // use the statusCode sent by the handler or use 200 as default
      statusCode = typeof(statusCode) === 'number' ? statusCode : 200;
      console.log('statusCode', statusCode);
      // use the payload sent by the handler or return an empty object
      payload = typeof(payload) === 'object' ? payload : {};
      console.log('payload', payload);
      // convert the payload to string as we are sending string from this request
      const payloadString = JSON.stringify(payload);
      console.log('returning this response: ', statusCode, payloadString);
      // Return the statuscode
      res.writeHead(statusCode);
      res.end(payloadString); 
    });
    // Outputs
    /* querystrings { one: 'hello', two: 'wow' }
    headers:  { 'content-type': 'application/x-www-form-urlencoded',
      headerone: 'haha',
      headertwo: 'this',
      headerthree: 'cool',
      'cache-control': 'no-cache',
      'postman-token': '7296ce06-2e17-41ec-8cab-0428784f920e',
      'user-agent': 'PostmanRuntime/7.1.5',
      accept: '/*',
      host: 'localhost:3000',
      'accept-encoding': 'gzip, deflate',
      'content-length': '41',
      connection: 'keep-alive' }
    payload name=Ravi&hobby=basketball&passion=coding
    method POST */
  });
});

// Start the server, and have it listen on port 3000
server.listen(3000, () => console.log('listening on port 3000'));

const handlers = {
  sample: (data, cb) => {
    // callback a http status code and a payload
    cb(406, {'name': 'Sample Handler'});
  },
  notFound: (data, cb) => {
    cb(404);
  }
};
const router = {
  'sample': handlers.sample,
};
```
### Setup routing.

* We will route according to the path requested by the user.
* We must setup a router that can match the incoming path requests to its respective handler.
* Any requests which doesn't have a match will be routed to the 404 handler.

* We define router with a route -- `sample` and the handler that runs when the route is hit `handlers.sample`. All other routes will be routed to the notFoundHandler which we don't need to write in the router object.
* The data in the handlers is the data that we have parsed from the request.
* The callback runs when we are done with handling the requests and tells us two things: a http status code and a payload object.
* Now we have our router and handler. We now need to modify our http server to send the data and recieves the cb data from that handler and to send the correct response code to the client.
* So we handle the route from the request on the `end` event which runs for every request.
* Choose the router handler and construct the data to be sent.
* Route the request to the handler specified in the router.
* In the chosenHandler, send in the data and the callback function. The data and the cb will go into the handler function and return to the callback of chosenHandler for further modifications.  

```js
const http = require('http');
const url = require('url');
const StringDecoder = require('string_decoder').StringDecoder;
const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const trimmedPathName = path.replace(/^\/+|\/+$/g, '');
  const queryStringObject = parsedUrl.query;
  const method = req.method;
  const headers = req.headers;
  const decoder = new StringDecoder('utf-8'); // we will get the utf-8 format of payload
  let buffer = '';
  req.on('data', (data) => {
    buffer += decoder.write(data);
  });
  req.on('end', () => {
    buffer += decoder.end(); // end the appending to buffer variable
    const chosenHandler = typeof(router[trimmedPathName]) !== 'undefined' ? router[trimmedPathName] : handlers.notFound;
    const data = {
      trimmedPathName,
      queryStringObject,
      headers,
      payload: buffer,
      method
    }; 
    chosenHandler(data, (statusCode, payload) => {
      statusCode = typeof(statusCode) === 'number' ? statusCode : 200;
      console.log('statusCode', statusCode);
      payload = typeof(payload) === 'object' ? payload : {};
      const payloadString = JSON.stringify(payload);
      res.writeHead(statusCode);
      res.end(payloadString); 
    });
  });
});
server.listen(3000, () => console.log('listening on port 3000'));
const handlers = {
  sample: (data, cb) => {
    // callback a http status code and a payload
    cb(406, {'name': 'Sample Handler'});
  },
  notFound: (data, cb) => {
    cb(404);
  }
};
const router = {
  'sample': handlers.sample,
};
```