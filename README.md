

## MongoDB Aggregation and Indexing

### `$match` , `$project` aggregation stage

Simple get all document of a collection we can use `find` query 

like: 
```js
db.test.find({})
```

We can do some thing using MongoDB `aggregation framework`

Example: 
```js
db.test.aggregate([])
```

#### 39. `$match`
Using `$match` stage we can set condition, and conditionally find document that we need for aggregation. 

Example:
```js
db.test.aggregate([
    // Stage 1
   {$match: {gender: "Male", age: {$lte: 25}}}

])
```

#### 40. `$project`
Using `$project` stage we can set the properties of the document we want to see in output. 

Example:
```js
db.test.aggregate([
    // Stage 1
   {$match: {gender: "Male", age: {$lte: 25}}},
   
   // Stage 2
   {$project: {name: 1, gender: 1}}

])

// Now this aggregation will show the document name and gender which full filled the condition of the $match stage 
```




### `$addFields` , `$out` , `$merge` aggregation stage

#### 41. `$addFields`
Using `$addFields` we can add field into aggregation but it doesn't add field to the main collection. 

Example: 
```js
db.test.aggregate([
    //-------Stage 1
    {$match: {gender: 'Male'}},
    
    //-------Stage 2
    {$addFields: {course: 'Next Level Web Development', edTech: 'Programming Hero'}},
    
    // ------Stage 3
    {$project: {email: 1, gender: 1, course: 1, edTech: 1}}
    
    ])
```

#### 42. `$out`
Using `$out` query `stage` we can create a new collection with the aggregation data. 

> Syntax: `{$out: "output-collection-Name"}`

Example:
```js
db.test.aggregate([
    //-------Stage 1
    {$match: {gender: 'Male'}},
    
    //-------Stage 2
    {$addFields: {course: 'Next Level Web Development', edTech: 'Programming Hero'}},
    
    // ------Stage 3
    {$project: {email: 1, gender: 1, course: 1, edTech: 1}},
    
    // -------Stage 4
    {$out: "courseStudents"}
    
    ])
```

#### 43. `$merge`
`$merge` stage will marge aggregation output the existing actual collection

> Syntax: ` {$merge: "existing collection name"}`

Example: 
```js
db.test.aggregate([
    //-------Stage 1
    // {$match: {gender: 'Male'}},
    
    //-------Stage 2
    {$addFields: {course: 'Next Level Web Development', edTech: 'Programming Hero'}},
    
    // ------Stage 3
    {$project: {email: 1, gender: 1, course: 1, edTech: 1}},
    
    // -------Stage 4
    {$merge: "test"}
    
    ])
```

### `$group` , `$sum` , `$push` aggregation stage

#### 44. `$group`
Using `$group` operator in this staging we can create document group based on specific Data of the collection document.
In this `$group` we can  find counts, totals, averages or maximums of document group. 

> `{$group: {_id: '$value-we-want-to-create-group-based-on'}}` *Here `_id` is mandatory*

Example:
```js
db.test.aggregate([
     //-------Stage 1
    {$group: {_id: '$address.country'}}
])

//or 
db.test.aggregate([
     //-------Stage 1
    {$group: {_id: '$age'}}
])

//or

db.test.aggregate([
     //-------Stage 1
    {$group: {_id: '$gender'}}
])
```

#### 45. `$sum`
Using `$sum` we can see total document count of the `$group`

Example:
```js
db.test.aggregate([
     //-------Stage 1
     {$group: {_id: '$gender', count: {$sum: 1}}}
])
```

#### 46. `$push`
In `$group` stage we can add properties into the group using `$push` operator. Using `$push` we can 
add data as a array property. 

Example of adding single value:
```js
db.test.aggregate([
     //-------Stage 1
     {$group: 
     {_id: '$gender', 
     count: {$sum: 1},
     userInfo: {$push: "$name.firstName"},
         
     }}
])
```

Example of adding all properties of document:
```js
db.test.aggregate([
     //-------Stage 1
     {$group: 
     {_id: '$gender', 
     count: {$sum: 1},
     userAllInfo: {$push: "$$ROOT"},
         
     }}
])
```

Example of adding all properties of document but getting limited by field filtering:

```js
db.test.aggregate([
     //-------Stage 1
     {$group: 
     {_id: '$gender', 
     count: {$sum: 1},
     userAllInfo: {$push: "$$ROOT"},
         
     }},

     //-------Stage 2
     {$project: {
     "userAllInfo.gender": 1,
     "userAllInfo.name": 1,
     "userAllInfo.email": 1,
     "userAllInfo.phone": 1,
     }}

])
```
### explore more about `$group` & `$project`

#### 47. `$group` operation `$sum`, `$max` , `$min` , `avg`

`$sum` use for see total value
`$max` use for find largest value
`$min` use for find smallest value
`$avg` use for see average of a value 

Example: 
```js
db.test.aggregate([
    // -------Stage 1
    {
        $group: {
            _id: null,
            totalDocument: {$sum: "$salary"} ,
            maximumSalary: {$max: "$salary"},
            minimumSalary: {$min: "$salary"},
            averageSalary: {$avg: "$salary"}

        }
    },
])

```
#### 48. `$project`
Besides field filtering we can do more with `$project` stage

Example:
```js
db.test.aggregate([
    // -------Stage 1
    {
        $group: {
            _id: null,
            totalDocument: {$sum: "$salary"} ,
            maximumSalary: {$max: "$salary"},
            minimumSalary: {$min: "$salary"},
            averageSalary: {$avg: "$salary"}

        }
    },
    
    // --------Stage 2 
    {
        $project: {
            // Rename Property in output
            totalDoc: "totalDocument",
            maxSalary: "$maximumSalary",
            minSalary: "$minimumSalary",
            avgSalary: "$averageSalary",
            
            // Operation in project 
            rangeBetweenMinAndMax: {$subtract: ["$maximumSalary", "$minimumSalary"]}
            
        }
    }
    ])
```

### Explore `$group` with `$unwind` aggregation stage

#### 49. `$unwind`

`$unwind` is a array method. Using `$unwind` we can break array and create multiple document with the array property. 
It helps to crate `group` with the array element. 

Example:
```js
db.test.aggregate([
    // -------Stage 1
   {$unwind: "$interests"},
    
    // --------Stage 2 
    {$group: { _id: "$age"  , interestPerAge: {$push: "$interests"}}}
   
])

```

#### 50. `$group` more
`$group` with array properties

Example:
```js
db.test.aggregate([
    // -------Stage 1
   {$unwind: "$interests"},
    
    // --------Stage 2 
    {$group: { _id: "$age"  , interestPerAge: {$push: "$interests"}}}
   
])

```
### `$bucket`, `$sort`, and `$limit` aggregation stage

#### 51. `$bucket`
Using `$bucket` we create a boundary where we can do special `group` operations. 

> `$bucket` syntax 
```js
{
  $bucket: {
      groupBy: <expression>,
      boundaries: [ <lowerbound1>, <lowerbound2>, ... ],
      default: <literal>,
      output: { 
         <output1>: { <$accumulator expression> },
         ...
         <outputN>: { <$accumulator expression> }
      }
   }
}
``` 

Example: 

```js
db.test.aggregate([
    // -------Stage 1
   {
       $bucket: {
           groupBy: "$age",
           boundaries: [20, 40, 60, 80],
           default: "total UpTo 80s are" , 
           output: {
               totalPeople: {$sum: 1},
               peopleWithAge: {$push: "$name.firstName"}
           }
       }
   },
    
   
])

```

#### 52. `$sort`
Using `$sort` stage in aggregation we can sort document in `Ascending` or `Descending` order. 

Example of `Ascending sort`:

```js
db.test.aggregate([
    // -------Stage 1
   {
       $bucket: {
           groupBy: "$age",
           boundaries: [20, 40, 60, 80],
           default: "total Upto 80 are" , 
           output: {
               totalPeople: {$sum: 1},
               peopleWithAge: {$push: "$name.firstName"}
           }
       }
   },
   
   // -----------Stage 2
   {
       $sort: {totalPeople: 1}
   },
    
   
])
```

Example of `Descending sort`:

```js
db.test.aggregate([
    // -------Stage 1
   {
       $bucket: {
           groupBy: "$age",
           boundaries: [20, 40, 60, 80],
           default: "total Upto 80 are" , 
           output: {
               totalPeople: {$sum: 1},
               peopleWithAge: {$push: "$name.firstName"}
           }
       }
   },
   
   // -----------Stage 2
   {
       $sort: {totalPeople: -1}
   },
    
   
])
```

#### 53. `$limit`
The `$limit` stage in aggregation use for define maximum output documents. 

Example: 
```js
db.test.aggregate([
    // -------Stage 1
   {
       $bucket: {
           groupBy: "$age",
           boundaries: [20, 40, 60, 80],
           default: "total Upto 80 are" , 
           output: {
               totalPeople: {$sum: 1},
               peopleWithAge: {$push: "$name.firstName"}
           }
       }
   },
   
   // -----------Stage 2
   {
       $sort: {totalPeople: 1}
   },
    
   // ------------Stage 3
   {
       $limit: 2
   },
])

```

### `$facet`, multiple pipeline aggregation stage

#### 54. `$facet`
Using `$facet` we can do multiple pipeline in a aggregation

Example:
```js
 db.test.aggregate([
    
    {
        $facet: {
            // ======== Pipeline  1
            "friendsCount": [
                // -----Stage 1
                {$unwind: "$friends"},
                
                // -----Stage 2
                {$group: { _id: "$friends", count: {$sum: 1}}}
                ],
                
            // ======== Pipeline  2
            "educationCount": [
                // -----Stage 1
                {$unwind: "$education"},
                
                // -----Stage 2
                {$group: { _id: "$education", count: {$sum: 1} }}
                ],
            
            // ======== Pipeline  3
            "skillsCount": [
                 // -----Stage 1
                 {$unwind: "$skills"},
                 
                 // -----Stage 2
                {$group: { _id: "$skills", count: {$sum: 1} }}
                ],
                

        }
    }
    
    
    ])

```

### `$lookup` stage, `embedding` vs `referencing`

#### 55. `embedding`
`Embedding` In this approach, related data is stored together in a single document. Like user product purchase info stored
 into the `user` document. 


#### 56. `referencing`
`Referencing` Referenced documents store relationships by including a reference (usually an ObjectId) to another document stored in a different collection. Like user product purchase info stored into another `purchase` collection using reference of `User Id`.


#### 57. `$lookup`
`$lookup` stage use for getting reference data form related document. 

Syntax:
```js
db.orders.aggregate([
     {
         $lookup: {
                from: "<collection name that I want to get data>",
                localField: "<The Present collection field name which based on I will do relation>",
                foreignField: "<The collection field name form where I want to get data>",
                as: "<output array field>"
              }
     }
    ])
```

Example:

````js
db.orders.aggregate([
     {
         $lookup: {
                from: "test",
                localField: "userId",
                foreignField: "_id",
                as: "userOrderList"
              }
     }
    ])
````

### What is `indexing`, `COLLSCAN` vs `IXSCAN`

#### 58. `indexing`
Document দ্রুত খুঁজে বের করার জন্য document এর কোন property উপর ভিত্তি করে index করে বা সাজিয়ে রাখাকে indexing বলে। 
আমারা Explicitly indexing না করেলেও Implicitly  `_id` উপর index হয়েই থাকে। 

Syntax for crating new Index:
> `db.test.createIndex({email:1})`

Syntax for delete an Index:
> `db.test.dropIndex({email:1})`

*[Note: We have to ignore multiple indexing. More indexing / multiple indexing consumes more memory]*

Example of how to see search time: 

```js
db.test.find({_id : ObjectId("6406ad63fc13ae5a40000065")}).explain("executionStats")
```

#### 59. `COLLSCAN`
When without any indexing we search a data it reads total document then it called `COLLSCAN`. It takes more time. 

#### 60. `IXSCAN`
When we search data using index property's value it called `IXSCAN`


### Explore `compound index` and `text index`

#### 61. `compound index`
We also create index using multiple properties, it called compound index. 

Example:

```js
db.bigData.createIndex({gender:1 , age: 1})

```

#### 62.`text index`
We can crate text index based on a `text property`

Example: based on `about` field create `text` index

```js
db.bigData.createIndex({about: "text"})
```

Example of search of this `text index`

```js
db.bigData.find({ $text: {$search: "nisi"}})
// nisi is the example of text value

```
