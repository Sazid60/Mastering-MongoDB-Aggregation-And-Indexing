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
