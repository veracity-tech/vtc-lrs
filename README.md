# Veracity LRS Lite

This project is intended to be a fully conformant xAPI implementation that can be integrated into other web projects. The module is published as Express middleware that can be simply integrated into an Express application. 

Data is stored in a MongoDB database.

The module can also run as a standalone webserver to accept xAPI statements. 

## Veracity Learning
[Veracity](https://veracity.it/) provides a more comprehensive and powerful learning platform: [Veracity Learning](https://lrs.io). Veracity Learning 
offers features such as [ADL's xAPI Launch](https://github.com/adlnet/xapi-launch), lesson uploads, and learning analytics. [Sign up for a free account](https://lrs.io/ui/users/create/) today.

## Installation

`npm install vtc-lrs`

## Changes
This release represents a major upgrade to the capabilities of the module, and is informed by the millions of xAPI statements stored by our production servers. 

1. Support for clients that mark statements as content-type=text/plain
1. Support for clients that mis-represent that post invalid JSON to the document endpoints
1. Support in general for many common errors related to inaccurate HTTP content-type headers
1. Loose conformance mode - accept misformed statements in many formats, including legacy TinCan style
1. Accept some variations in conformance in the query - allow short format dates, double encoded JSON agent queries
1. Aggregation and Advanced Query APIs

### Breaking Changes from Version 1
All updates above are compatible. The only change to the API is that the 'statementStored' event now sends an array, instead of a single string.


## Use

### As Express middleware

#### Using the API 
This is the simplest possible use. This endpoint will accept any basic auth credentials, and use the default configuration.
```
const xapi = require('vtc-lrs');
app.use("/xapi", new xapi() );

```

The middleware can also be configured with several options.
```
const xapi = require('vtc-lrs');

let options = {
  lrs: new xapi.LRS(),  
  connectionString:"mongodb://localhost",
  getUser:function(req, username, password) {
    return new xapi.Account(username,true,true);
  },
  baseUrl: "http://localhost:3000/xapi"
};
app.use("/xapi", new xapi(options) );


```

Each setting in the options object is optional and can be omitted. 

`lrs` 

An instance of the xapi.LRS constructor. This controls how data is stored and provides the API. Note that the API is not connected to the database until this object is passed into the middleware constructor. This object will expose the API for integration.


`connectionString`

A string representing the connection to the MongoDB server. 

`baseUrl`

The public facing pathname to the API, including the domain. This is required so that the API can generate the MORE links correctly.  

`getUser`

This function takes the request, the username, and the password, in that order, as parameters. The function must return either null, in the case that the username/password pair should not be accepted by the API, or an xapi.Account object, in the case that the username is accepted. You can also provide a promise to an Account. The account constructor has 3 inputs, like below

```
...
getUser:function(req, username, password) {
    return new xapi.Account(username, //The name of the agent for the Authority
      true,  //this account has read access to the API
      true,   //this account has write access to the API
      false,   //this account can only retreive data it posted 
      true,   //this account can use the advanced search apis
    ); 
  },
...
```

### Responding to events

The API will dispatch events on the LRS object. The current events supported are "statementStored", "clientError'", and "ready".  
```
  
  let lrs = new xapi.LRS(); 
  
  let XAPI = new xapi( {
    lrs: lrs,
    getUser: getUser,
    connectionString: process.env.connectionString || "mongodb://localhost/bcc5ce8c-0e47-4863-a43a-7a6f8928c2a8",
    baseUrl: process.env.host || "http://localhost:3000/xapi"
  } );
  //a batch of statements was stored. A single POSTed statement is treated as an array with one item  
  lrs.on("statementStored", function(ids)
  {
    console.log("The IDs of the stored statements are " + ids.join(","));
    lrs.getStatement(ids[0]).then(statement => {
      console.log(statement);
    })
  })

  lrs.on("clientError", function(e)
  {
    console.log("The client sent an invalid xAPI request");
  })

  lrs.on('ready', function(){
    console.log("The lrs is attached to the database and ready. You can now use the programmatic API.")

    lrs.insertStatement(someValidJSON).then( ()=>{
      console.log("The statement was stored");
    }).catch( (e)=>{
      console.log("There was some problem with the statement.", e)
    })
  })

```

#### Using the UI

The VTC-LRS also comes with a very simple UI. You can install this using the code below.

```
const XAPI = require('vtc-lrs');
let xapi = new XAPI(settings);
app.use("/xapi", xapi );
app.use("/ui", xapi.simpleUI());
```

The connection string must be a string to a MongoDB database. You can also provide a function that takes the Express request as the first parameter, and returns a connection string or a promise to one. 

### As a standalone LRS server.

If you want to simply run an LRS, without integrating it into your own Express application, you can simple run the module. You'll have to navigate into the directory with the code. If you've cloned the repo from GitHub, you can just run the commands below. If you've used NPM to install the module, navigate into the directory `./node_modules/vtc-lrs/` before running the commands below.

`npm start` or `node index.js -start` will launch the server with default configuration. `node index.js -start -loose` will run the server in loose conformance mode, which will fail the test suite but support TinCan and other common error cases. To override the defaults, you can create a file called `.env` in the directory alongside index.js. This file must use the format:

```
connectionString=mongodb://localhost/myLrs
port=3000
apiPath=/xapi
host=http://localhost:3000
uiPath=/ui
username=basicApiUser
password=basicApiPassword
```

All .env file parameters are optional, with the defaults set as above. In the case where username and password are omitted from the .env file, the API will accept any credentials. You can access the xapi at http://localhost:3000/xapi, and the SimpleUI at http://localhost:3000/ui.
