# MongoDB Setup

## Test Setup

Creating Docker Env (with MongoDB):
```
sudo docker run --name test-mongo -dit -p 27017:27017 --rm mongo
```

To list docker images running:
```
sudo docker ps
```

To run the docker image:
```
sudo docker exec -it test-mongo mongo
```

## MongoDB commands

To list the all the databases:
```
show dbs;
```

To create a database:
```
use <database name>;
```
i.e.
```
use adoption;
```

To show the current working database:
```
db;
```

To create a collection:
(the following query will create `pets` collection on the fly)
```
db.pets.createOne({name: "Luna", type: "dog", breed: "Havanese", age: 8});
```

To insert many data in our database: (use `insertMany()`)
```db.pets.insertMany(
  Array.from({ length: 10000 }).map((_, index) => ({
    name: [
      "Luna",
      "Fido",
      "Fluffy",
      "Carina",
      "Spot",
      "Beethoven",
      "Baxter",
      "Dug",
      "Zero",
      "Santa's Little Helper",
      "Snoopy",
    ][index % 9],
    type: ["dog", "cat", "bird", "reptile"][index % 4],
    age: (index % 18) + 1,
    breed: [
      "Havanese",
      "Bichon Frise",
      "Beagle",
      "Cockatoo",
      "African Gray",
      "Tabby",
      "Iguana",
    ][index % 7],
    index: index,
  }))
);
```
###Projections
```
db.pets.find({ type: "dog" }, { name: 1, breed: 1, _id: 0 });
db.pets.find({ type: "dog" }, { name: true, breed: true, _id: false }); // note that true and false work too
```

### Updates
#### updateOne
```
db.pets.updateOne(
  { type: "dog", name: "Luna", breed: "Havanese" },
  { $set: { owner: "Peter" } }
);
```
#### updateMany
```
db.pets.updateMany({ type: "dog" }, { $inc: { age: 1 } });
```

### Upsert (update and insert)
```
db.pets.updateOne(
  {
    type: "dog",
    name: "Sudo",
    breed: "Wheaten",
  },
  {
    $set: {
      type: "dog",
      name: "Sudo",
      breed: "Wheaten",
      age: 5,
      index: 10000,
      owner: "Sarah Drasner",
    },
  },
  {
    upsert: true,
  }
);
```

### Delete
`db.pets.deleteMany({ type: "reptile", breed: "Havanese" })`

### Indexes in MongoDB

To get the indexes:
```
db.pets.getIndexes();
```

### Create Index:
```
db.pets.createIndex({ name: true });
```

### Create Unique Index:
```
db.pets.createIndex({ index: 1 }, { unique: true });
```

To check execution status
```
db.pets.find({ name: "Fido" }).explain("executionStats");
```

### Text Index

#### Creating Text Index
```
db.pets.createIndex({
  type: "text",
  breed: "text",
  name: "text",
});
```
### Text Index Search
```
db.pets
  .find({ $text: { $search: "dog Havanese Luna" } })
  .sort({ score: { $meta: "textScore" } });
```

### Text Index Searching and Sorting
```
db.pets
  .find(
    { $text: { $search: "dog Havanese Luna" } },
    { score: { $meta: "textScore" } }
  )
  .sort({ score: { $meta: "textScore" } });
```

If you want to search for all Lunas that are not cats you can do this:
```
db.pets
  .find({ $text: { $search: "-cat Luna" } })
  .sort({ score: { $meta: "textScore" } });
```

## Aggregation

If we wanted to know how many puppies, adult, and senior dogs we have in our pets collection? Let's try just that
```
db.pets.aggregate([
  {
    $bucket: {
      groupBy: "$age",
      boundaries: [0, 3, 9, 15],
      default: "very senior",
      output: {
        count: { $sum: 1 },
      },
    },
  },
]);
```
If all the want is dogs to aggregate, then
```
db.pets.aggregate([
  {
    $match: {
      type: "dog",
    },
  },
  {
    $bucket: {
      groupBy: "$age",
      boundaries: [0, 3, 9, 15],
      default: "very senior",
      output: {
        count: { $sum: 1 },
      },
    },
  },
]);
```
Using the `$match` stage of the aggregation, we can exclude every pet that isn't a dog.

Last one, what if we wanted to sort the results by which group had the most pets?
```
db.pets.aggregate([
  {
    $match: {
      type: "dog",
    },
  },
  {
    $bucket: {
      groupBy: "$age",
      boundaries: [0, 3, 9, 15],
      default: "very senior",
      output: {
        count: { $sum: 1 },
      },
    },
  },
  {
    $sort: {
      count: -1,
    },
  },
]);
```



