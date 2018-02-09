# VTC LRS

This project is intended to be a fully conformant xAPI implementation that can be integrated into other web projects. The module is published as Express middleware that can be simply integrated into an Express application. 

Data is stored in a MongoDB database.

The module can also run as a standalone webserver to accept xAPI statements. 

## Veracity Learning
Veracity provides a more comprehensive and powerful learning platform: [Veracity Learning](https://lrs.io). Veracity Learning 
offers features such as [ADL's xAPI Launch](https://github.com/adlnet/xapi-launch), lesson uploads, and learning analytics. [Sign up for a free account](https://lrs.io/ui/users/create/) today.

## Installation

`npm install vtc-lrs`

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
  baseUrl: "/xapi"
};
app.use("/xapi", new xapi(options) );


```

Each setting in the options object is optional and can be omitted. 

`lrs` 

An instance of the xapi.LRS constructor. This controls how data is stored and provides the API. Note that the API is not connected to the database until this object is passed into the middleware constructor. This object will expose the API for integration.


`connectionString`

A string representing the connection to the MongoDB server. 

`baseUrl`

The public facing pathname to the API, not including the domain. This is required so that the API can generate the MORE links correctly.  

`getUser`

This function takes the request, the username, and the password, in that order, as parameters. The function must return either null, in the case that the username/password pair should not be accepted by the API, or an xapi.Account object, in the case that the username is accepted. You can also provide a promise to an Account. The account constructor has 3 inputs, like below

```
...
getUser:function(req, username, password) {
    return new xapi.Account(username, //The name of the agent for the Authority
      true,  //this account has read access to the API
      true   //this account has write access to the API
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
    baseUrl: process.env.baseUrl || "/xapi"
  } );

  lrs.on("statementStored", function(id)
  {
    console.log("The ID of the stored statement is " + id);
    lrs.getStatement(id).then(statement => {
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

`npm start` or `node index.js -start` will launch the server with default configuration. To override the defaults, you can create a file called `.env` in the directory alongside index.js. This file must use the format:

```
connectionString=mongodb://localhost/myLrs
port=3000
baseUrl=/xapi
uiPath=/ui
username=basicApiUser
password=basicApiPassword
```

All .env file parameters are optional, with the defaults set as above. In the case where username and password are omitted from the .env file, the API will accept any credentials. You can access the xapi at http://localhost:3000/xapi, and the SimpleUI at http://localhost:3000/ui.
