# clev-cqrs

clev-cqrs is a nodejs library that allows you to incorporate CQRS and Event sourcing to your nodejs project without any stress. it is compatible with any database and also designed to work in asynchronous environment. clev-cqrs surports callbacks. Just hearing CQRS or Event Sourcing for the first time? check out [this](https://eventstore.com/blog/event-sourcing-and-cqrs/) article

## Documentation

The official documentation website is at our[repository](https://github.com/Giftedcooperation/clev-cqrs).

## Supported Database

clev-cqrs supports any database. All relational(sql) and non-relational(no-sql) database is well supported

## Contributors

Pull requests are always welcome! Please base pull requests against the `master`
branch and follow the [contributing guide](https://github.com/Giftedcooperation/clev-cqrs/CONTRIBUTING.md).
Please do **not** modify any files in the `public`, `express`and `views` folder. Please make
your changes in `src` folder.

## Installation

First install [Node.js](http://nodejs.org/). Then:

```sh
$ npm install clev-cqrs
```

## Importing

```javascript
// Using Node.js `require()`
const clevCqrs = require("clev-cqrs");

// Using ES6 imports
import clevCqrs from "clev-cqrs";
```

## Overview

### Create a Schema

First, we need to create a schema using the Schema constructor from clev-cqrs, the schema should look something like this

```js
let   clevCqrs = require("clev-cqrs");
const ExampleView2 = require("../views/ExampleView2");
const ExampleView = require("../views/ExampleView");
const eventStore = require("../eventstore/eventstore");
let SchemaName = new clevCqrs.Schema({
  schemaName: "schemaName e.g Product",
  fields: {
    //fields here
    //    e.g   name:{
    //          type:'string',
    //          required:true
    //          },
  },
  commandHandlers: {
    //handleCreate command handler is required
    handleCreate: function (yourEntityName, eventData) {
      eventData.id = yourEntityName.id; //required
      // add other entity details here using example of this format
      // eventData.description=yourEntityName.description
      // eventData.name = yourEntityName.name
    },

    // other command handlers here e.g
    // handleChangeName: function(product, eventData){
    //     eventData.name= product.name
    // },
  },
  eventStore: {
    methods: {
      get: eventStore.get,
      delete: eventStore.delete,
      getAll: eventStore.getAll,
      save: eventStore.save,
    },
  },
  eventHandlers: {
    views: [ExampleView, ExampleView2],
  },
});

clevCqrs.setSchema(SchemaName);
```

Your schema should contain the following

### schemaName `string`

This should be the name of your business entity e.g Product, User e.t.c

### fields `object`

This should contain all the property of the entity, each property should have its own `type` and `required` field e.g

```js
    fields: {
        name:{
         type:'string',// all types would be discussed later on this page
         required:true // can also be false
        },

    },
```

### commandHandlers `object`

commandHandlers are mearnt to handle any change that happens to your entity e.g(change of name, price, age and so on), each handler points to a function that takes in two parameters, `1` (eventData) which is the current state of your entity and `2`An object with the changes you made e.g {name:'Roberto Callus'} you can then set the changes to your eventData as shown in the code below.


Note that the `handleCreate` command handler is required as this would be used to create your entity and don't forget to set the id property(id would be autogenerated you don't need to border about it just set it that's all). Also every command handler must start with the word `handle` e.g `handleDebitWallet`, `handleShipOrder` and every handler is mearnt to handle a change in each property i.e handleChangeName should only handle change of name, handleUpdateAge should only handle change in age e.t.c

```js
    commandHandlers: {
        //handleCreate command handler is required
        //every command handler must start with handle
        handleCreate: function (changesObject, eventData) {
            eventData.id = changedObject.id //required
            // add other entity details here using example of this format
            // eventData.description= changedObject.description
            // eventData.name = changedObject.name
        },


        handleChangeName: function(changesObject, eventData){
            eventData.name= changesObject.name
        },
        handleDebitWallet: function(changesObject, eventData){
            eventData.wallet = eventData.wallet - changesObject.amount
        },

        //....

    }

```

### eventStore `object`

the event store is an object that points to methods. There are four methods that must be provided
`get`, `getAll`, `delete` and `save`. The `get` property should be set to an asynchronous function that takes in an id, search for an object from your database with the id and return it. The `getAll` property should be set to an asynchronous function that returns all the documents in your database in as an array.The `delete` property should be set to an asynchronous function that takes in an id and delete a document based on the id that was passed. The `save` property should be set to a function that saves a object to your database. e.g

```js
//if you are using a mongo db
    eventStore: {
            methods: {
                        get: async function (id) {
                            let event = await db.collection('events').findOne({ id })
                            return event
                        },
                        delete: async function (id) {
                              await db.collection('events').deleteOne({ id })
                        },
                        getAll: async function () {

                            let events = await db.collection('events').find({})
                            let eventsToArray = await events.toArray()
                            return eventsToArray
                        },
                        save: async function (event) {
                           await db.collection('events').insertOne(event)

                        }
        }
    },
```

### eventHandlers `object`

event handlers is an object that points to different views that you want in your service.

```js
eventHandlers: {
  views: [ExampleView, ExampleView2];
}
```

### views `object`

views are various ways you want see your data represented, for example if you are working on an e-commerce service, you can have a view that displays all the order that has been placed so far (`ordersView`) and another view that displays the customer that has placed the highest number of orders(`cutomerPeformanceView`). Each view respond to events that concerns it, lets use the example above, each order would have properties like
`orderPlacedBy`, `items`,`totalPrice`, `shippingAddress`, `totalQuantity`. When the `shippingAddress` is updated, only the `ordersView` would be affected but if the `orderPlacedBy` was updated both the `ordersView` and the `cutomerPeformanceView` would be updated.

###

To create a view, create an instance of the clevCqrs.View then execute the setHandlers method. the setHandlers method takes in an object parameter, this object would have the handlers for any change that happens, each handler is a function that takes in two parameters, `1` an object representing the change that was made e.g {age : 20} . `2` callback this should always be called after event is handled to show that no error occured, you can pass an error to it, it would be thrown if it occurs. Note that eventHandlers in a view must use thesame name that was used for the commandHandler e.g `handleChangeName` , `handleChangeAddress` and so on. Each view must have a handleCreate event handler as this would be used to create the entity don't forget to set the id property on the entity while creating. An example of a view is shown below

```js
const clevCqrs = require("clev-cqrs");
let ProductDetailsView = new clevCqrs.View();
//set handler for any event any event that is related to this view, this should relate to changes you want to make to the read database
// for example if this a ProductDetails view you might want to check for events that affect all attribute of the product and if it's a ProductPrice view you might want to check for event that affect only name and price attribute
ProductDetailsView.setHandlers({
  handleCreate: async function (product, callback) {
    let itemToAddTODB = {
      id: product.id, // required
      name: product.name,
      description: product.description,
      size: product.size,
      price: 2000,
    };
    // add itemToAddTODB to database

   db.collection("products").insertOne(itemToAddTODB).then(()=>{
       callback(null)
   })
   .catch((err)=>{
       callback(err);
   })
    // return a callback to show there are no errors  -required-
    // required
  },


  // add other event handlers here for example
  handleChangeName: async function (product, callback) {
    // update database
    await db.collections("products").updateOne({ id: product.id }, {$set:{name:product.name}}).then(()=>{
       callback(null)
   })
   .catch((err)=>{
       callback(err);
   })
  },


    handleChangePrice: async function (product, callback) {
    // update database
    await db.collections("products").updateOne({ id: product.id }, {$set:{price:product.price}}).then(()=>{
       callback(null)
   })
   .catch((err)=>{
       callback(err);
   })
  },
});

module.exports = ExampleView2;
```



### setSchema method

once you are done creating the schema which include all the proprties explained above jist call the setSchema function passing the schema that you created

```js
clevCqrs.setSchema(SchemaName);
```

# sending a command
After you have created a schema and set it, the steps remaining are very few. Call the module where you created your schema in your root module e.g (app.js or index.js) by using the require function. You can send commands by using the clevCQRS.handler object.the format is clevCQRS.handler.`command handlers name e.g handleCreate or handleChangeName`. The commandHandler that you provided in your schema would be used here. Note that every command handler apart from the handleCreate takes in an id `string`, command `object` and callback function,the id is the id of document that you want to change , command would take in the name of the attribute that you want to change and the new value you want it to be, and the callback would be a function the returns the updated entity and  all the previous events that has ever occured to that entity. e.g

```js

let express = require('express')
const app = express()
require('./schema')
const clevCqrs = require('clev-cqrs')
app.use(clevCqrs.generatePage)

app.post('/create', (req,res)=>{
    let command = {
        name:req.body.name,
        description:req.body.description,
        price:req.body.price,
        size:req.body.size
   }
   clevCqrs.handler.handleCreate(command,function(updatedevent){
    res.send({created: updatedevent.id})
   })
   
    
    
})

app.post('/description/:id', (req,res)=>{
    let command = {
        description:req.body.description,
    }
    clevCqrs.handler.handleChangeDescription(req.params.id, command,function(updatedevent){
     if(event){
        res.send({done:updatedevent})
     }
    })
  
})

```


## eventstore page

clev-cqrs provides a nice user interface to view all the events that has occured to all you data. This include the event id, the event type e.g(changeName), the event time and the event data. to generatge this page just add the clevCQRS.generatePage to your middleware. e.g
```js
const clevCqrs = require('clev-cqrs')
app.use(clevCqrs.generatePage)
```


## Types in clev-CQRS
clev-Cqrs support various types wish are
 * string
 * number
 * buffer
 * date
 * mix
 * object
 * array

The types are used when contructing your schema. An example of how to use the types is in the code below

```js
let   clevCqrs = require("clev-cqrs");
const ExampleView2 = require("../views/ExampleView2");
const ExampleView = require("../views/ExampleView");
const eventStore = require("../eventstore/eventstore");
let SchemaName = new clevCqrs.Schema({
  schemaName: "User",
   fields: {

    name: {
      type: 'string',
      required: true
    },
    age: {
      max: 100,
      min: 1,
      type: 'number'
    },
    nameToBuffer: {
      type: 'buffer'
    },
    dob: {
      type: 'date',
      required: false
    },
    mixed: {
      type: 'mix'
    },
    address: {
      type: 'object'
    },
    roles: {
      type: 'array'
    }

  },
  commandHandlers: {
    //handleCreate command handler is required
    handleCreate: function (yourEntityName, eventData) {
      eventData.id = yourEntityName.id; //required
      // add other entity details here using example of this format
      // eventData.description=yourEntityName.description
      // eventData.name = yourEntityName.name
    },

    // other command handlers here e.g
    // handleChangeName: function(product, eventData){
    //     eventData.name= product.name
    // },
  },
  eventStore: {
    methods: {
      get: eventStore.get,
      delete: eventStore.delete,
      getAll: eventStore.getAll,
      save: eventStore.save,
    },
  },
  eventHandlers: {
    views: [ExampleView, ExampleView2],
  },
});

clevCqrs.setSchema(SchemaName);
```

```
   


## License

Copyright (c) 2020 Adewumi Sunkanmi;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
