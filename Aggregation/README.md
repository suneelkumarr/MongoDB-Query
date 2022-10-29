# AGGREGATION

DATASET:

```js
{
        "_id" : ObjectId("603a87095854104ef6c863e1"),
        "gender" : "male",
        "name" : {
                "title" : "mr",
                "first" : "zachary",
                "last" : "lo"
        },
        "location" : {
                "street" : "3193 king st",
                "city" : "chipman",
                "state" : "yukon",
                "postcode" : "H8N 1Q8",
                "coordinates" : {
                        "latitude" : "76.4507",
                        "longitude" : "-70.2264"
                },
                "timezone" : {
                        "offset" : "+11:00",
                        "description" : "Magadan, Solomon Islands, New Caledonia"
                }
        },
        "email" : "zachary.lo@example.com",
        "login" : {
                "uuid" : "76970c67-4801-4926-80f0-4872fe0aee42",
                "username" : "lazyrabbit189",
                "password" : "pass1",
                "salt" : "BVMLMPwZ",
                "md5" : "a6ff61f912af9958587e0fc0c8dc920b",
                "sha1" : "bd37d1c699fb5a17031924c37e5d90ba4403e598",
                "sha256" : "0305e3ebf6f4502790d804cff5989a6a928f466af6e36bd808ad9ed24e51fee7"
        },
        "dob" : {
                "date" : "1988-10-17T03:45:04Z",
                "age" : 29
        },
        "registered" : {
                "date" : "2011-09-29T20:54:32Z",
                "age" : 6
        },
        "phone" : "273-427-0510",
        "cell" : "309-911-7770",
        "id" : {
                "name" : "",
                "value" : null
        },
        "picture" : {
                "large" : "https://randomuser.me/api/portraits/men/9.jpg",
                "medium" : "https://randomuser.me/api/portraits/med/men/9.jpg",
                "thumbnail" : "https://randomuser.me/api/portraits/thumb/men/9.jpg"
        },
        "nat" : "CA"
}
```

Aggregation basically groups the data from multiple
documents and operates in many ways on those grouped data in order to
return one combined result.

Unlike Refs/Populate, aggregation runs the query on the server side (mongodb) and returns the processed documents.

It works with STAGES:

`STAGE1` ==(altered_data)==> `STAGE2` ==(altered_data)==> `STAGE3`

Also,  *it works perfectly with indexes similar to how "find works"* .

Query:

```js
// takes in an array (pipeline)
db.contacts.aggregate([
  // $match === exactly same as find
  { $match: { gender: 'female' } },
]);
```

Here $match is a stage (of a pipeline which decide the returning docs). We can add multiple stages.

Let us now try and

**QUESTION**
Find a custom collection that tells us how many people are living in all the states with the state names.

```js
db.contacts.aggregate([
  { $match: { gender: 'female' } }, // stage1
  {
    $group: {
      // accumulates data
      // stage2
      _id: { stateName: '$location.state' }, // _id is a syntax here
      totalPeople: { $sum: 1 }, // aggregation functions (applied on all the returned docs (eg: $sum) )
    },
  },
]);
```

Retreived Data:

```js
{ "_id" : { "stateName" : "hautes-alpes" }, "totalPeople" : 2 }
{ "_id" : { "stateName" : "paris" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "bremen" }, "totalPeople" : 11 }
{ "_id" : { "stateName" : "oregon" }, "totalPeople" : 7 }
{ "_id" : { "stateName" : "åland" }, "totalPeople" : 10 }
{ "_id" : { "stateName" : "antalya" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "brandenburg" }, "totalPeople" : 12 }
{ "_id" : { "stateName" : "kastamonu" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "eskişehir" }, "totalPeople" : 3 }
{ "_id" : { "stateName" : "dorset" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "mecklenburg-vorpommern" }, "totalPeople" : 11 }
{ "_id" : { "stateName" : "cumbria" }, "totalPeople" : 2 }
{ "_id" : { "stateName" : "melilla" }, "totalPeople" : 5 }
{ "_id" : { "stateName" : "قم" }, "totalPeople" : 4 }
{ "_id" : { "stateName" : "uşak" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "burdur" }, "totalPeople" : 2 }
{ "_id" : { "stateName" : "gelderland" }, "totalPeople" : 16 }
{ "_id" : { "stateName" : "arizona" }, "totalPeople" : 4 }
{ "_id" : { "stateName" : "dordogne" }, "totalPeople" : 1 }
{ "_id" : { "stateName" : "bingöl" }, "totalPeople" : 2 }
```

Let us try and sorting the above result:

```js
db.contacts.aggregate([
  { $match: { gender: 'female' } }, // stage1
  { $group: { _id: { state: '$location.state' }, totalPersons: { $sum: 1 } } }, // stage2
  { $sort: { totalPersons: -1 } }, // stage3
]);
```

---

## Project stage

Helps us transform the returning documents:

**QUESTION**

Let us say we want to:

* Get all the docs with age > 20
* I don't want IDS to show
* I only want their genders to show
* I want a new field "fullName" with their first and last names concatinated

```js
db.contacts.aggregate([
  { $match: { 'dob.age': { $gt: 20 } } },
  { $project: { _id: 0, gender: 1, fullName: { $concat: ['$name.first', ' ', '$name.last'] } } },
]);
```

```js
db.contacts.aggregate([
        {$match: {'dob.age': {$gt: 20}}},
        {$project: {
            _id: 0,
            gender: 1,
            fullName: {
                $concat: [
                        { $toUpper: { "$name.first" } },
                        " ",
                        { $toUpper: { "$name.last" } }
                ]
           ` }
        }}
]);
```

Let us say, we want only the first alphabets to be in the uppercase.

```js
db.contacts.aggregate([
  { $match: { 'dob.age': { $gt: 20 } } },
  {
    $project: {
      _id: 0,
      gender: 1,
      fullName: {
        $concat: [
          { $toUpper: { $substrCP: ['$name.first', 0, 1] } },
          {
            $substrCP: [
              '$name.first',
              1, // start cutting from first index
              { $subtract: [{ $strLenCP: '$name.first' }, 1] }, // until length -1
            ],
          },
          ' ',
          { $toUpper: { $substrCP: ['$name.last', 0, 1] } },
          { $substrCP: ['$name.last', 1, { $subtract: [{ $strLenCP: '$name.first' }, 1] }] },
        ],
      },
    },
  },
]);
```

## Multi stage PROJECT

```js
db.contacts.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      email: 1, // make sure you inherit fields at every stage to show them
      location: {
        random: 'field',
        coordinates: ['$location.coordinates.longitude', '$location.coordinates.latitude'],
      },
    },
  },
  {
    $project: {
      location: 1,
      email: 1, // make sure you iherit field
      fullName: {
        $concat: ['$name.first', ' ', '$name.last'],
      },
    },
  },
]);
```

Returned documents:

```js
[
  {
    location: { random: 'field', coordinates: ['101.5995', '78.8545'] },
    email: 'isolino.viana@example.com',
    fullName: 'isolino viana',
  },
  {
    location: { random: 'field', coordinates: ['-18.5996', '-42.6128'] },
    email: 'elijah.lewis@example.com',
    fullName: 'elijah lewis',
  },
];
```

Same example as above, but this time we want the coordinates to be in integers:

```js
db.contacts.aggregate([
  {
    $project: {
      _id: 0,
      name: 1,
      location: {
        random: 'field',
        coordinates: [
          {
            $convert: {
              input: '$location.coordinates.longitude',
              to: 'double',
              onError: 0.0, // in case some error occurs, return as 0.0
              onNull: 0, // in case its null, return 0.0
            },
          },
          {
            $convert: {
              input: '$location.coordinates.latitude',
              to: 'double',
              onError: 0.0,
              onNull: 0,
            },
          },
        ],
      },
    },
  },
  {
    $project: {
      location: 1,
      fullName: {
        $concat: ['$name.first', ' ', '$name.last'],
      },
    },
  },
]);
```

---

 **question** :

I want the collection of all the users who's age is greater than 25 and print their names and their *dob.date* to  *generic date type* .

```js
db.contacts.aggregate([
  {
    $match: { 'dob.age': { $gt: 25 } },
  },
  {
    $project: {
      _id: 0,
      name: {
        $concat: ['$name.first', ' ', '$name.last'],
      },
      birthdate: {
        $convert: {
          input: '$dob.date',
          to: 'date', // predefined mongo date type
          onError: 0.0,
          onNull: 0,
        },
      },
    },
  },
]);
```

**Result**

```js
{ "name" : "zachary lo", "birthdate" : ISODate("1988-10-17T03:45:04Z") }
{ "name" : "louise graham", "birthdate" : ISODate("1971-01-21T20:36:16Z") }
{ "name" : "harvey chambers", "birthdate" : ISODate("1988-05-27T00:14:03Z") }
{ "name" : "victor pedersen", "birthdate" : ISODate("1959-02-19T23:56:23Z") }
```

 **SHORTCUT** :

```js
db.contacts.aggregate([
  {
    $match: { 'dob.age': { $gt: 25 } },
  },
  {
    $project: {
      _id: 0,
      name: {
        $concat: ['$name.first', ' ', '$name.last'],
      },
      birthdate: { $toDate: '$dob.date' }, // toDate is another aggregate operator
    },
  },
]);
```

## DATASET (ADVANCE)

```js
[
  {
    "projectName": "first",
    "resources": [
      {
        "resource": "EC2",
        "region": "ap-south-1",
        "params": {
          "ImageId": "ami-0bcf5425cdc1d8a85",
          "InstanceType": "t2.micro"
        }
      },
      {
        "resource": "S3",
        "region": "ap-south-1",
        "params": {
          "Bucket": "test-bucket"
        }
      }
    ]
  }
]
```

I want to find out the documents who's resources array have 'EC2' and print out ONLY that element from the array.

**Solution**

```js
const resources = await Project.aggregate([
     {
       $match: {
         resources: {
           $elemMatch: {
             resource: { $eq: resourceType },
           },
         },
       },
     },
     {
       $project: {
         projectName: '$projectName',
         createdAt: '$createdAt',
         resources: {
           $filter: {
             input: '$resources',
             as: 'each', // use $$ to refer
             cond: { $eq: ['$$each.resource', 'EC2'] },
           },
         },
       },
     },
   ]);
```

OR

* $filter to iterate loop of resources array and find matching resource
* $arrayElemAt to get first matching element from above filtered result
* $replaceRoot to replace above return object to root

```js
const projects = await Project.aggregate([

  { $match: { resources: { $elemMatch: { resource: { $eq: "EC2" } } } } },
  // or below match is equal to above match condition
  // { $match: { "resources.resource": "EC2" } },

  {
    $replaceRoot: {
      newRoot: {
        $arrayElemAt: [
          {
            $filter: {
              input: "$resources",
              cond: { $eq: ["$$this.resource", "EC2"] }
            }
          },
          0
        ]
      }
    }
  }
])
```
