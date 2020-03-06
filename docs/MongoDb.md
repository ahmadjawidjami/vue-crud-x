# Mongo Atlas

## Setup

1. User & Password

2. IP Whitelist

## NodeJS Connection

const MongoClient = require('mongodb').MongoClient;
const uri = "mongodb+srv://<user>:<password>@<cluster-name>-<random-5-letters>.gcp.mongodb.net/test?retryWrites=true&w=majority";
const client = new MongoClient(uri, { useNewUrlParser: true });
client.connect(err => {
  const collection = client.db("test").collection("devices");
  // perform actions on the collection object
  client.close();
});

## Robomongo / Robo3T Connection

1. Goto a cluster and find primary shard

<cluster-name>-shard-00-01-<random-5-letters>.gcp.mongodb.net:27017

2. Connection Tab

- Type: Direct Connection

- Name: <something-meaningful>

- Address: <cluster-name>-shard-00-01-<random-5-letters>.gcp.mongodb.net

- Port: 27017

3. Authentication Tab

- Perform Authentication: ticked

- Database: admin

- User Name:

- Password:

- Auth Mechanism: SCRAM-SHA-1 / 256 (usually 1)

4. SSL Tab

- Use SSL protocol: ticked

- Authentication Method: Self-signed Certificate




# reconnection

1. If there is no connection initially, connection failure will not result in reconnection

2. does happen, to test out. shutdown mongo server, wait awhile then start up. CRUD will take effect and updates will take effect also...


# Change Streams

https://docs.mongodb.com/manual/changeStreams/

## watch changes

in mongo client, run rs.initiate() - but how to start mongo rs without calling this command?

```
.\bin\mongod.exe --dbpath .\bin\data --port 27017 --replSet rs0
```

```js
mongo.stream = db.db('mm').collection('exchangeUsers').watch()
    mongo.stream.on('change', (change) => {
    console.log(change); // You could parse out the needed info and send only that data.
    // use websocket to listen to changes
})
```

## replace

```js
{
  _id: { 
    _data: '825D07B9C7000000012B022C0100296E5A10046D79825AAFE447EE93D29D46B0D1AF5846645F696400645CE20EDD520C352CBCF453E80004'
  },
  operationType: 'replace',
  clusterTime: Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1560787399 },
  fullDocument: {
    _id: 5ce20edd520c352cbcf453e8,
    username: 'user5a',
    email: 'user5',
    clientId: '5c9b1eb8cb3521eccb35431b',
    password: '$2a$12$1K.MWGt6Ez30WJ05mYAK6u/dvjoce0O.IATL2TYxLGxCv2rju9ccS',
    gaKey: 'IZDXCUDYNQ4ESMZZNY4HGZSDJRAVGZCO',
    sms: '',
    smsLastSent: null,
    smsOtpPin: '',
    smsVerified: 0,
    telegramId: '',
    telegramUsername: '',
    role: 'user'
  },
  ns: { db: 'mm', coll: 'exchangeUsers' },
  documentKey: { _id: 5ce20edd520c352cbcf453e8 }
}
```

## insert

```js
{ _id:
   { _data:
      '825D07BBBC000000012B022C0100296E5A10046D79825AAFE447EE93D29D46B0D1AF5846645F696400645D07BBBCCAABD31DCDB648950004' },
  operationType: 'insert',
  clusterTime:
   Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1560787900 },
  fullDocument:
   { _id: 5d07bbbccaabd31dcdb64895,
     username: 'user6aa',
     email: 'user6aa',
     clientId: '5c9b1eb8cb3521eccb35431b',
     password:
      '$2a$12$1K.MWGt6Ez30WJ05mYAK6u/dvjoce0O.IATL2TYxLGxCv2rju9ccS',
     gaKey: 'IZDXCUDYNQ4ESMZZNY4HGZSDJRAVGZCO',
     sms: '',
     smsLastSent: null,
     smsOtpPin: '',
     smsVerified: 0,
     telegramId: '',
     telegramUsername: '',
     role: 'user' },
  ns: { db: 'mm', coll: 'exchangeUsers' },
  documentKey: { _id: 5d07bbbccaabd31dcdb64895 } }
```

## delete

```js
{ _id:
   { _data:
      '825D07BBEF000000012B022C0100296E5A10046D79825AAFE447EE93D29D46B0D1AF5846645F696400645D07BBBCCAABD31DCDB648950004' },
  operationType: 'delete',
  clusterTime:
   Timestamp { _bsontype: 'Timestamp', low_: 1, high_: 1560787951 },
  ns: { db: 'mm', coll: 'exchangeUsers' },
  documentKey: { _id: 5d07bbbccaabd31dcdb64895 }
}
```

```json
{
  "_id":{
  "_data":"825C5D51F70000000129295A1004E83608EE8F1B4FBABDCEE73D5BF31FC946645F696400645C5D51F73ACA83479B48DE6E0004"},
  "operationType":"insert",
  "clusterTime":"6655565945622233089",
  "fullDocument":{
  "_id":"5c5d51f73aca83479b48de6e",
  "ticker":"AAPL",
  "Price":300
  },
  "ns":{"db":"mydb","coll":"Stocks"},
  "documentKey":{"_id":"5c5d51f73aca83479b48de6e"}
}
```

# TRANSACTIONS

https://docs.mongodb.com/manual/core/transactions/

```js
  const session = client.startSession();
  const transactionOptions = {
    readPreference: 'primary',
    readConcern: { level: 'local' },
    writeConcern: { w: 'majority' }
  };
  try {
    await session.withTransaction(async () => {
      // Important:: You must pass the session to the operations
      await client.db('mydb1').collection('foo').insertOne({ abc: 1 }, { session });
      await client.db('mydb2').collection('bar').insertOne({ xyz: 999 }, { session });
    }, transactionOptions);
  } finally {
    await session.endSession();
    await client.close();
  }
```


## Some Useful Queries

db.getCollection('dd_book').aggregate([ { $lookup: {from: "dd_author", localField:"author", foreignField:"_id", as: "in_common" }}])

db.getCollection('dd_book').aggregate([
  { $lookup: {from: "dd_author", localField:"authorCode", foreignField:"code", as: "author_info" }},
  { $lookup: {from: "dd_cat", localField:"catCode", foreignField:"code", as: "cat_info" }}
])

// in pipeline{ $replaceRoot: { newRoot: "$date" } },

db.getCollection('dd_book').aggregate([
  {
    $lookup: {
      from: "dd_author",
      let: { author_code: "$authorCode" },
      pipeline: [
        {
          $match: { 
            $expr: { $eq: [ "$code",  "$$author_code" ] },
          }
        },
        { $project: { _id: 0, code: 0 } }
      ],
      as: "author_details"
    }
  },
  {
    $lookup: {
      from: "dd_cat",
      let: { cat_code: "$catCode" },
      pipeline: [
        {
          $match: { 
            $expr: { $eq: [ "$code",  "$$cat_code" ] },
          }
        },
        { $project: { _id: 0, code: 0 } }
      ],
      as: "cat_details"
    }
  }
]) 



db.getCollection('dd_author').insertMany([
  {
    "_id" : ObjectId("5cdb5249525a06edd9634e1f"),
    "name" : "aaron",
    "code" : "aa"
  },
  {
    "_id" : ObjectId("5cdb5258525a06edd9634e23"),
    "name" : "ben",
    "code" : "bb"
  }
])

db.getCollection('dd_cat').insertMany([
  {
    name: "cat1",
    code: "c1"
  },
  {
    name: "cat2",
    code: "c2"
  }
])

db.getCollection('dd_book').insertMany([
  {
      "_id" : ObjectId("5cdb5282525a06edd9634e2f"),
      "name" : "book a",
      "author" : ObjectId("5cdb5249525a06edd9634e1f"),
      "authorCode" : "bb",
      "catCode:": "c1"
  },
  {
      "_id" : ObjectId("5cdb52a0525a06edd9634e36"),
      "name" : "book b",
      "author" : ObjectId("5cdb5258525a06edd9634e23"),
      "authorCode" : "bb",
      "catCode:": "c2"
  },
  {
      "_id" : ObjectId("5cdb5359525a06edd9634e4e"),
      "name" : "book c",
      "author" : ObjectId("5cdb5249525a06edd9634e1f"),
      "authorCode" : "bb",
      "catCode:": "c1"
  }
])