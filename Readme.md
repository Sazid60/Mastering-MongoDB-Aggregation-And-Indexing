# Mastering-MongoDB-Aggregation-And-Indexing

Practice Data: https://github.com/Apollo-Level2-Web-Dev/mongodb-practice

In This Module, You'll Be Introduced To The Robust Aggregation Framework In MongoDB. Gain An Understanding Of How This Framework Empowers Advanced Data Processing And Manipulation, Providing A Flexible And Efficient Approach To Data Analysis Within MongoDB.

What Will You Learn From This Module?

0. Introduction of a powerful aggregation framework: Dive into the world of data manipulation with the powerful Aggregation Framework. We'll explore how it allows you to process, group, and summarize data from your MongoDB collections.

1. $match , $project aggregation stage: Dive into the $match and $project aggregation stages. Learn how $match filters documents based on specified criteria, while $project enables reshaping by including or excluding fields.

2. $addFields , $out , $merge aggregation stage:Explore the $addFields, $out, and $merge aggregation stages. Understand how to add new fields, write results to a new collection overwrite an existing one, and merge results into an existing collection.

3. $group , $sum , $push aggregation stage: Delve into the $group aggregation stage, allowing grouping by a specified key. Learn to use $sum to calculate the sum within a group and $push to create arrays of values.

4. explore more about $group & $project: Further, explore the capabilities of $group and $project aggregation stages. Understand advanced techniques in grouping and reshaping data for specific analysis requirements.

5. Explore $group with $unwind aggregation stage: Discover the $group aggregation stage combined with $unwind. Learn how to handle arrays within documents, facilitating more complex aggregations.

6. $bucket, $sort, and $limit aggregation stage: Explore the $bucket aggregation stage for categorizing data, $sort for sorting results, and $limit for limiting document output.

7. $facet, multiple pipeline aggregation stage: Dive into the $facet aggregation stage, enabling the execution of multiple pipelines within a single stage. Understand how this feature facilitates parallel processing of various aggregations.

8. $lookup stage, embedding vs referencing: Explore the $lookup aggregation stage for performing left outer joins between collections. Understand the concepts of embedding and referencing for efficient data modeling.

9. What is indexing, COLLSCAN vs IXSCAN: Learn about indexing in MongoDB, its importance in optimizing query performance, and the distinction between COLLSCAN (Collection Scan) and IXSCAN (Index Scan).

10. Explore compound index and text index: Delve into compound indexes, involving multiple fields, and text indexes designed for efficient text searching. Understand how these indexes enhance query efficiency.

Embark on this journey to master MongoDB Aggregation and enhance your data manipulation and analysis skills within the MongoDB environment! Happy learning!

## 16-0 Intro the powerful aggregation framework

![alt text](<WhatsApp Image 2025-06-12 at 11.28.57_4908e22b.jpg>)

- Aggregation Is The way of processing a large number of documents in a collection by means of passing them through different stages.
- The stages make up what is known as pipeline
- The stages in a pipeline can filter, sort, group, reshape and modify documents that passes through the pipeline
- The aggregation pipeline is often preferred and the recommended way of doing aggregations in MongoDB. It is designed specifically to improve performance and usability for aggregation. Pipeline operators need not produce one output document for every input document, but can also generate new documents or filter out documents. Moreover, starting from MongoDB version 4.4, it can also define custom aggregation expressions with $accumulator and $function.

#### Lets Understand with an example

- Suppose You have 8 cousins. You are planning to do tour. Before the tour date suppose some have exam, some arises with financial issues and some became sick

![alt text](<WhatsApp Image 2025-06-12 at 11.35.47_cc1c1f19.jpg>)

- Just 3 Cousins are left. Here Big Brother will manage all tour related things. For this `Sort By Age` will be done to make the big brother the leader.
- Suppose Before the Tour Date Another Occurrence happened like you just got 2 ticket. The problem is there is no chance to change the tour date. So we have do `limit by two` and remove the younger one.

![alt text](<WhatsApp Image 2025-06-12 at 11.40.07_14880330.jpg>)

- Then the two cousins are put in a `group` and calculated the budget.

![alt text](<WhatsApp Image 2025-06-12 at 11.41.57_08c9d88b.jpg>)

- here, sor, limit, group is done in different stages. These are called aggregation.

#### Syntax Of Aggregation

```js
db.collection.aggregate([
  // stage-1
  {}, //---->pipeline
  // stage-2
  {}, //---->pipeline
  // stage-3
  {}, //---->pipeline
]);
```

![alt text](<WhatsApp Image 2025-06-12 at 11.45.25_8219daa0.jpg>)

- One Stage Will Pass the data to another stage.

![alt text](<WhatsApp Image 2025-06-12 at 11.47.12_83e35b72.jpg>)

```js
db.cousins.aggregate([
  // filter out the cousins who have exam
  { $match: { hasExam: { $ne: true } } },

  // Filter out cousins who have a budget less than 5000
  { $match: { budget: { $gte: 5000 } } },

  // filter out cousins who is sick
  { $match: { isSick: false } },

  // sort by age
  { $sort: { age: -1 } },

  // limit by 2
  { $limit: 2 },

  // calculate budget
  {
    $group: {
      _id: "null",
      totalBudget: { $sum: "$budget" },
      cousins: { $push: "$name" },
    },
  },
]);
```

#### Most Commonly Used Aggregation are

- $project: Reshapes each document in the stream, e.g., by adding new fields or removing existing fields. For each input document, output one document.
- $match: Filters the document stream to allow only matching documents to pass unmodified into the next pipeline stage. For each input document, the output is either one document (a match) or zero document (no match).
- $group: Groups input documents by a specified identifier expression and apply the accumulator expression(s), if specified, to each group. $group consumes all input documents and outputs one document per each distinct group. The output documents only contain the identifier field (group id) and, if specified, accumulated fields.
- $sort: Reorders the document stream by a specified sort key. The documents are unmodified, except for the order of the documents. For each input document, the output will be one document.
- $skip : Skips the first n documents where n is the specified skip number and passes the remaining documents unmodified to the pipeline. For each input document, the output is either zero document (for the first n documents) or one document (after the first n documents).
- $limit : Passes the first n documents unmodified to the pipeline where n is the specified limit. For each input document, the output is either one document (for the first n documents) or zero document (after the first n documents).
- $unwind : Breaks an array field from the input documents and outputs one document for each element. Each output document will have the same field, but the array field is replaced by an element value per document. For each input document, outputs n documents where n is the number of array elements and can be zero for an empty array.

## 16-1 $match , $project aggregation stage

[Mongodb Aggregation](https://studio3t.com/knowledge-base/articles/mongodb-aggregation-framework/)

![alt text](image.png)

#### $match stage

- The $match stage allows us to choose just those documents from a collection that we want to work with. It does this by filtering out those that do not follow our requirements.

- aggregate is similar to find

```js
db.test.find({});
```

```js
db.test.aggregate([]);
```

##### Implementing single condition

```js
db.test.find({ gender: "Male" });
```

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male" } },
]);
```

- We can Implement Multiple condition

```js
db.test.find({ gender: "Male", age: { $lt: 30 } });
```

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male", age: { $gt: 30 } } },
]);
```

#### $project stage

- In MongoDB, the $project aggregation stage is used to shape the structure of the documents that result from a pipeline. Same as Field Filtering

```js
db.test
  .find({ gender: "Male", age: { $lt: 30 } })
  .project({ name: 1, gender: 1, age: 1 });
```

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male", age: { $gt: 30 } } },

  // stage-2
  { $project: { name: 1, age: 1, gender: 1 } },
]);
```

- Lets Make a twist here

```js
db.test.aggregate([
    // stage-1
        {$project:{name:1,gender:1}}

    // stage-2

    { $match: { gender: "Male", age:{$gt : 30 } }},
])
```

- This will not show any document (empty array) since first stage is not passing the age to the second stage but there is work related to age and which is under implicit and condition.

- For this reason we will use project at the end of all stages so that no hassle occurs.

## 16-3 $addFields , $out , $merge aggregation stage

```js
db.test.aggregate([
  // Stage-1: Filter for males
  { $match: { gender: "Male" } },

  // Stage-2: Filter for age <= 30
  { $match: { age: { $lte: 30 } } },

  // Stage-3: Project only selected fields
  { $project: { gender: 1, age: 1, name: 1 } },

  // Stage-4: Sort by age in ascending order (change 1 to -1 for descending order)
  { $sort: { age: 1 } },
]);
```

- The more we use stages it will take more time. I mean It Will Extend the Query Time.
- Our Target should be like we will use less stages so that Query Time Reduces.

#### $addfields Stage

- As the $addFields documentation points out , The added fields only apply to the document in the context of pipeline
- That means the original document is not modified.
- You can add $addfields at any point in the pipeline, deriving fields from the data in the pervious stage.

- If we want to add new filed with the existing field we have to use $addField. It will not modify the original document, it will just add a field in the pipeline.
- It will be used when the situation is like add a new field to the data and give me so that i can add in new collection, we will use this. (for adding in new collection we have to use $$out stage as well)

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male", age: { $gt: 30 } } },
  // Stage-2
  { $addFields: { course: "Level-2", eduTech: "Programming Hero" } },
  // stage-3
  { $project: { course: 1, eduTech: 1 } },
]);
```

- It will not add to the original document it will just show adding the new data.

#### #out stage

- If we want to add new fields and create a new collection with the added fields we have to use $out stage
- This is an unusual type of stage because it allows you to carry the results of your aggregation over into a new collection, or into an existing one after dropping it, or even adding them to the existing documents.
- The $out stage must be the last stage in the pipeline.

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male", age: { $gt: 30 } } },
  // Stage-2
  {
    $addFields: {
      course: "Level-2",
      eduTech: "Programming Hero",
      monerMoto: "Moner Iccha",
    },
  },
  // stage-3
  //   { $project: { course: 1, eduTech: 1 } },

  // stage-4

  { $out: "Course-Students" },
]);
```

#### $merge stage

- If we want to add new fields and merge with the existing collection we have to use $merge
- It Will Not Create New Collection but It will add the mentioned data in the existing collection i mean it will marge stage

```js
db.test.aggregate([
  // stage-1
  { $match: { gender: "Male", age: { $gt: 30 } } },
  // Stage-2
  {
    $addFields: {
      course: "Level-2",
      eduTech: "Programming Hero",
      monerMoto: "Moner Iccha",
    },
  },
  // stage-3
  { $merge: "test" },
]);
```

## 16-3 $group , $sum , $push aggregation stage

#### $group stage

- With the $group stage, we can perform all the aggregation or summary queries that we need, such as finding counts, totals, averages or maximums.
- Divides into multiple bases doing grouping.
- It is responsible for grouping and summarizing documents. It takes multiple documents and arranges them into several separate batches based on grouping.

```js
db.test.aggregate([
  // stage-1
  { $group: { _id: "$gender" } },

  // stage-2
]);
```

- \_id in $group is required and determines how documents are grouped. It can be any field, a computed value, or even null (to group all documents together).

- Before gender, the $ is used to refer to a field in the document. So, $gender means "take the value of the gender field" from each document. In the $group stage, _id: "$gender" means we are grouping documents by the value of the gender field — it's used as the grouping key.

```js
db.test.aggregate([
  // stage-1
  { $group: { _id: "$address.country" } },

  // stage-2
]);
```

| **Operator** | **Meaning**                                                         |
| ------------ | ------------------------------------------------------------------- |
| `$count`     | Calculates the quantity of documents in the given group.            |
| `$max`       | Displays the maximum value of a document’s field in the collection. |
| `$min`       | Displays the minimum value of a document’s field in the collection. |
| `$avg`       | Displays the average value of a document’s field in the collection. |
| `$sum`       | Sums up the specified values of all documents in the collection.    |
| `$push`      | Adds extra values into the array of the resulting document.         |

#### $group with $sum

- If we want to count the distinct group values we have to use $sum with the $group

```js
db.test.aggregate([
  // stage-1
  { $group: { _id: "$address.country", count: { $sum: 1 } } },
]);
```

![alt text](image-1.png)

- This will sum all the document under one country group
- Here `count: { $sum: 1 } }` is `accumulator` and the name can be count, total or anything.

#### $group with $push

- Adds extra values into the array of the resulting document.

- This will additionally add the names who are with the country groups and count the persons

```js
db.test.aggregate([
  // stage-1
  {
    $group: {
      _id: "$address.country",
      totalPolapanInCountry: { $sum: 1 },
      polapanErName: { $push: "$name" },
    },
  },
]);
```

- The $push operator adds the value of the name field from each document into an array (polapanErName) for each group.

- If we want to show all the fields we have to use `$$ROOT`

```js
db.test.aggregate([
  // stage-1
  {
    $group: {
      _id: "$address.country",
      totalPolapanInCountry: { $sum: 1 },
      polapanErFullDoc: { $push: "$$ROOT" },
    },
  },
  //   stage-2
  {
    $project: {
      "polapanErFullDoc.name": 1,
      "polapanErFullDoc.email": 1,
      "polapanErFullDoc.phone": 1,
    },
  },
]);
```

## 16-4 explore more about $group & $project

- Suppose we want to make the entire data set as group. we have to use `_id:null`. This makes entire collection a group and helps to do accumulator operations like `$sum, $avg, $min,$max,$count etc` in entire collection.

```js
db.test.aggregate([
  // stage-1
  {
    $group: {
      _id: null,
      totalSalary: { $sum: "$salary" },
      maxSalary: { $max: "$salary" },
      minSalary: { $min: "$salary" },
      avgSalary: { $avg: "$salary" },
    },
  },
]);
```

#### Renaming Inside `$project`

```js
db.test.aggregate([
  // stage-1
  {
    $group: {
      _id: null,
      totalSalary: { $sum: "$salary" },
      maxSalary: { $max: "$salary" },
      minSalary: { $min: "$salary" },
      avgSalary: { $avg: "$salary" },
    },
  },

  // stage-2

  {
    $project: {
      totalSalary: 1,
      maxSalary: 1,
      minSalary: 1,
      averageSalary: "$avgSalary",
    },
  },
]);
```

- here we have changed the avgSalary to averageSalary

#### Calculation inside `$project` using `$subtract`

- Structure

```
{ $subtract: [ <expression1>, <expression2> ] }
```

- Suppose we want to see the gap between(difference) the maximum and minimum salary

```js
db.test.aggregate([
  // stage-1
  {
    $group: {
      _id: null,
      totalSalary: { $sum: "$salary" },
      maxSalary: { $max: "$salary" },
      minSalary: { $min: "$salary" },
      avgSalary: { $avg: "$salary" },
    },
  },

  // stage-2

  {
    $project: {
      totalSalary: 1,
      maxSalary: 1,
      minSalary: 1,
      averageSalary: "$avgSalary",
      rangeBetweenMaxAndMin: { $subtract: ["$maxSalary", "$minSalary"] },
    },
  },
]);
```

## 16-5 Explore $group with $unwind aggregation stage

#### $unwind with $group

- There are some problems with group
- Group can be done in single value
- We may have array and array of object this causes problem with $group
- This considers array as a distinct value, since we can not work directly on the element of an array within an array within a document with the stages such as $group.
- $unwind stage enables us to work with the value of the fields within an array
- Where there is an array field within the input document, you will sometimes need to output the document several times once for every element of that array.

##### why to use $unwind?

- You can not work directly on the elements of the array within a documents with stages like $group.
- $unwind stage enables us to work with the values of the fields with the array
- $unwind takes the array and and goes through each and every element of the array and makes individual groups.

![alt text](<WhatsApp Image 2025-06-13 at 12.22.08_d8be8b27.jpg>)

```js
db.test.aggregate([
  // stage-1
  { $unwind: "$friends" },
  // stage-2
  { $group: { _id: "$friends", count: { $sum: 1 } } },
]);
```

- It counts how many times each individual friend appears in the friends array across all documents in the test collection.

- This breaks down each array element into its own document.

- After unwinding, each friend in the friends array becomes a separate document.

- Groups all documents by the value of friends.

- Uses $sum: 1 to count how many times each friend appears.

#### Lets see another example of $unwind

- Suppose we have a situation like we have to group based on the age and then we have to figure out each groups Interests from the interests array.

```js
db.test.aggregate([
  // stage-1
  { $unwind: "$interests" },

  // stage-2
  {
    $group: { _id: "$age", interestsPerAge: { $push: "$interests" } },
  },
]);
```

## 16-6 $bucket, $sort, and $limit aggregation stage

- $bucket says the name itself that what is it. Its like keeping different ranges things in different buckets.

![alt text](<WhatsApp Image 2025-06-14 at 08.37.24_b6b6c06f.jpg>)

- In MongoDB, the $bucket aggregation stage is used to group documents into a specified number of ranges, or "buckets," based on the values of a specified field. It is particularly helpful for performing range-based data aggregation, similar to SQL's GROUP BY functionality but with custom numeric or date ranges.

- Structure

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

- Suppose we have a scenarios like we have to separate the persons in different ranges of age segment. after that we have to count them.

```js
db.test.aggregate([
  // Stage 1
  {
    $bucket: {
      groupBy: "$age", // The field used to group documents, in this case, the `age` field.
      boundaries: [20, 40, 60, 80], // Specifies the ranges (or buckets) for grouping.
      // This creates buckets: [20-40), [40-60), [60-80).
      // Each range is inclusive of the lower bound and exclusive of the upper bound.
      default: "80 er uporer buira gula", // Specifies a label for any values above the last boundary (80).
      // Any documents with age >= 80 will be put into this "default" bucket.
      output: {
        count: { $sum: 1 }, // Counts the number of documents in each bucket.
        karKarAse: { $push: "$$ROOT" }, // Pushes the full document (`$$ROOT`) of each document in the bucket
        // into an array called `karKarAse`.
      },
    },
  },

  // Stage 2
  { $sort: { count: -1 } }, // Sorts the output buckets in descending order by the `count` field.

  // Stage 3
  { $limit: 4 }, // Limits the output to only the top 4 buckets (based on the sorted count from the previous stage).

  // Stage 4
  { $project: { count: 1 } }, // Projects only the `count` field in the final output.
  // This will return only the count of each bucket, omitting the other fields.
]);
```

## 16-7 $facet, multiple pipeline aggregation stage

- The $facet stage in MongoDB's aggregation pipeline is used to perform multiple, parallel aggregations on the same set of documents, producing separate sub-pipelines that run independently. This is useful when you want to calculate multiple aggregations at once and get a combined result, allowing you to avoid running separate queries for each calculation.
- Facet is used to create multi pipeline.
- Its required when the situation is like if there is requirement of generating multiple report based on one single data.
- Sub pipelines under facet is not dependent to each other and will work parallel

![alt text](image-2.png)

```js
db.test.aggregate([
  {
    $facet: {
      // pipeline-1
      pipeline1: [
        // stage-1
        {
          $group: {
            _id: "$address.country",
            count: { $sum: 1 },
            stateName: { $push: "$address.city" },
          },
        },
        // stage-2
      ],
      // pipeline-2
      friendsCount: [
        // Stage-1
        { $unwind: "$friends" },
        // stage-2
        { $group: { _id: "$friends", count: { $sum: 1 } } },
      ],
      // Pipeline-3
      educationCount: [
        // Stage-1
        { $unwind: "$education" },
        // stage-2
        { $group: { _id: "$education", count: { $sum: 1 } } },
      ],
      // Pipeline-4
      skillsCount: [
        // Stage-1
        { $unwind: "$skills" },
        // stage-2
        { $group: { _id: "$skills", count: { $sum: 1 } } },
      ],
    },
  },
]);
```

## 16-8 $lookup stage, embedding vs referencing

![alt text](<WhatsApp Image 2025-06-14 at 10.36.33_ed85c3a1.jpg>)

- Suppose I am a user and now i will order a product from a website

![alt text](<WhatsApp Image 2025-06-14 at 10.37.29_aed7a089.jpg>)

- Product data will be embedded inside the user data.

**This is not appropriate option to do because ife the product purchase increases the embedding filed will increase and the data will become heavier. And this will increase the query time.**

- The appropriate way to do is referencing. Like creating a different field and then refer the two fields.

![alt text](<WhatsApp Image 2025-06-14 at 10.42.51_a6dab0ef.jpg>)

#### when To use referencing and embedding

![alt text](image-3.png)

| **Aspect**            | **Embedded**                | **Referencing**                            |
| --------------------- | --------------------------- | ------------------------------------------ |
| **Relationship Type** | One-to-one relationship     | One-to-many or many-to-many relationships  |
| **Best For**          | Frequently reading data     | Frequently writing or updating data        |
| **Updates**           | Atomic updates              | May require multiple updates               |
| **Network Overhead**  | Reduces network overhead    | Higher network overhead for large datasets |
| **Data Size**         | Suitable for small datasets | Scalable for large datasets                |
| **Flexibility**       | Less flexible               | Highly flexible                            |

#### Use Case Examples

##### Embedded Example:

- Embedding is ideal when you need to store a small, self-contained dataset, such as a user's profile information directly inside a parent document.

##### Referencing Example:

- Referencing works best for relational data, like associating a product with multiple categories or linking users to their orders.

#### What is $Lookup?

- In MongoDB, $lookup is an aggregation pipeline stage used to perform joins between collections. It allows you to combine data from two collections, similar to SQL joins, by matching a field from one collection with a field from another. Its like it will look for the referenced data in another collection and provide us the data by merging with our data.

- structure

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "<collection to join>", // this means where/ which collection we will search i will find the data
      localField: "<field from the input documents>",
      foreignField: "<field from the documents of the from collection>",
      as: "<output array field>",
    },
  },
]);
```

#### Example

```js
db.orders.aggregate([
  {
    $lookup: {
      from: "test",
      localField: "userId",
      foreignField: "_id",
      as: "UserInformation",
    },
  },
]);
```

## 16-9 What is indexing, COLLSCAN vs IXSCAN

- Lets guess you have my name in a book. If you want to find in by going through line by line it is called COLLSCAN.
- COLSCAN Consumes some time to find a data. this is a problem
- If you create an index to find it more faster its called IXSCAN. Indexing means you have a content indexes and you can go directly to the desired page. When indexing is done the indexes will bge in sorted manner(asc or desc).
- When we do search a data it will search in the index and through the index searching is placed where the actual data exists. we do not need to go page by page or line by line. and query time reduces.
- To observe which type is used we can directly make query in mongodb compass or in noSqlBooster.
- Maximum Time we will search by id since it will use mongodb default indexing which will require less time

```js
db.test
  .find({ _id: ObjectId("6406ad63fc13ae5a40000067") })
  .explain("executionStats");
```

![alt text](image-4.png)

```js
db.test.find({ email: "omirfin2@i2i.jp" }).explain("executionStats");
```

![alt text](image-5.png)

- for massive data we should create indexing. Though we should not create much since more we make indexing more it takes memory
- We can create index in mongodb Compass as well we can do it by coding
- Indexing can be created in ascending or descending order. helps in sorting data

#### Ascending order index creating

```js
db.getCollection("massive-data").createIndex({ email: 1 });
```

## 16-10 Explore compound index and text index

#### Delete a Indexing

```js
db.getCollection("massive-data").dropIndex({ email: 1 });
```

#### Lets Understand Compound Indexing

- when we try to mach a document based on two different field we need compound indexing.

```js
{gender:"Male", age:21}
```

- If do this query it will show us that COLSCAN Is happening here.

![alt text](image-6.png)

- Here Order Matters. When we sort data we have do sorting of one field then another.
- mongodb Behind the scene it uses balanced tree.

[Balanced Tree](https://www.cs.usfca.edu/~galles/visualization/BTree.html)

![alt text](image-7.png)

![alt text](image-8.png)

- we can use command line to create as well

```js
db.getCollection("massive-data").createIndex({ gender: -1, age: 1 });
```

- here gender indexed ascending order taking into concern that Male is searched more than female. so Male will come first while searching.

#### Search Index

- This facilitates us to flexible the searching based n the text of any field. Its like it will not require to write full word to find anything.

- Creating a search index

```js
db.getCollection("massive-data").createIndex({ about: "text" });
```

![alt text](image-9.png)

Example for searching a text

```js
db.getCollection("massive-data")
  .find({ $text: { $search: "dolor" } })
  .project({ about: 1 });
```

- Which Filed Requires More Searching we will do Search Indexing.

## Practice Task

Practice Data: https://raw.githubusercontent.com/Apollo-Level2-Web-Dev/mongodb-practice/main/massive-data.json

Task Link: https://drive.google.com/file/d/14Bl2h_ctiAmVB-9Xvb0Z9DAuELfDhXv0/view?usp=drive_link

Solution: https://github.com/Apollo-Level2-Web-Dev/practice-tasks-2-solutions

![alt text](image-10.png)

1. Retrieve the count of individuals who are active (isActive: true) for each
   gender.

```js
db.getCollection("massive-data").aggregate([
  // stage-1
  { $match: { isActive: { $eq: true } } },
  // stage-2
  { $group: { _id: "$gender", count: { $sum: 1 } } },
]);
```

2. Retrieve the names and email addresses of individuals who are active
   (`isActive: true`) and have a favorite fruit of "banana".

```js
db.getCollection("massive-data").aggregate([
  // stage-1
  { $match: { isActive: true, favoriteFruit: "banana" } },
]);
```

3. Find the average age of individuals for each favorite fruit, then sort the
   results in descending order of average age.

```js
db.getCollection("massive-data").aggregate([
  // stage-1
  { $group: { _id: "$favoriteFruit", averageAge: { $sum: 1 } } },
  // stage-2
  { $sort: { averageAge: -1 } },
]);
```

4. Retrieve a list of unique friend names for individuals who have at least
   one friend, and include only the friends with names starting with the
   letter "W".

```js
db.getCollection("massive-data").aggregate([
  { $unwind: "$friends" },
  {
    $match: {
      "friends.name": /^W/,
    },
  },
  {
    $group: {
      _id: "$_id",
      uniqueFriends: { $addToSet: "$friends.name" },
    },
  },
]);
```

5. Use $facet to separate individuals into two facets based on their age:
   those below 30 and those above 30. Then, within each facet, bucket the
   individuals into age ranges (e.g., 20-25, 26-30, etc.) and sort them by
   age within each bucket.

```js
db.getCollection("massive-data").aggregate([
  {
    $facet: {
      below30: [
        { $match: { age: { $lt: 30 } } },
        {
          $bucket: {
            groupBy: "$age",
            boundaries: [20, 25, 30],
            default: "Other",
            output: {
              names: { $push: "$name" },
            },
          },
        },
        {
          $sort: { age: 1 },
        },
      ],
      above30: [
        { $match: { age: { $gte: 30 } } },
        {
          $bucket: {
            groupBy: "$age",
            boundaries: [30, 35, 40],
            default: "Other",
            output: {
              names: { $push: "$name" },
            },
          },
        },
      ],
    },
  },
  {
    $sort: { age: 1 },
  },
]);
```

6. Calculate the total balance of individuals for each company and display the company name along with the total balance. Limit the result to show only the top two companies with the highest total balance.

```js
db.getCollection("massive-data").aggregate([
  {
    $group: {
      _id: "$company",
      totalBalance: { $sum: { $toDouble: { $substr: ["$balance", 1, -1] } } },
    },
  },
  {
    $sort: { totalBalance: -1 },
  },
  {
    $limit: 2,
  },
]);
```
