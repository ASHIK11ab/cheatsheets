# Flask App (Python) - Mongo DB Cheatsheet

### Overview
- This cheatsheet will cover on connecting to Mongo DB from flask app and executing basic CRUD operations from python.

| &emsp;&emsp;Table of Contents &emsp;&emsp;|
| --- |
| 1. [Initial Setup in Flask](#initial-setup) |
| 2. [Creating Collections](#creating-collections) |
| 3. [Inserting Documents](#inserting-documents) |
| 4. [Selecting Documents](#selecting-documents) |
| 5. [Selecting documents with filters](#selecting-documents-with-filters) |
| 6. [Projecting specific fields](#projecting-specific-fields-in-a-collection) |
| 7. [Updating documents](#updating-documents) |
| 8. [Deleting documents](#deleting-documents) |

### Setting up the environment
- Clone this repository.
- Install the necessary dependencies by running
  ```bash
    > pipenv sync
  ```
- This will install the dependencies from Pipfile.lock

### Manual Installation
- Manually install the dependencies by running
```bash
  > pipenv install Flask Flask-PyMongo pymongo[srv]
```

### Initial setup
- Since we will be using <kbd>Flask-PyMongo</kbd> we will first create our <kbd>Flask</kbd> application and set up some initial configuration.

```py
  from flask import Flask

  app = Flask(__name__)
  app.config["MONGO_URI"] = "mongodb+srv://<username>:<password>@<uri>/<dbname>"
```
> Replace `username`, `password`, `uri`, `dbname` with your actual values.

2. Now, lets create our mongo instance and bind it to our Mongo DB database.
```py
  from flask_pymongo import PyMongo

  mongo = PyMongo(app)
```
> Flask-PyMongo will find the Mongo DB URI from our app's configuration.

3. Now, lets consolidate our initial setup.
```py
  from flask import Flask
  from flask_pymongo import PyMongo

  app = Flask(__name__)
  app.config["MONGO_URI"] = "mongodb+srv://<username>:<password>@<uri>/<dbname>"

  mongo = PyMongo(app)
```

### Creating collections
1. Now, lets create a collection called <kbd>users</kbd> in Mongo DB using our mongo instance.
1. The <kbd>db</kbd> object of <kbd>mongo</kbd> instance gives allows us to interact with the database.
1. Inorder to create a collection we will use the <kbd>db</kbd> object and the collection name.
1. Ex, <kbd>db.\<collection-name></kbd>
```py
  db = mongo.db

  # db.users will return the collection if the collection is present, if not it will create a new collection named `users`.
  users = db.users
```

### Inserting documents
1. Now that we have our <kbd>users</kbd> collection, lets insert some documents into it.
1. Documents in Mongo DB are similar to JSON objects or like dictionaries in Python.
1. A document will contain a list of keys and its associated value. An example document is given below.
```js
  {
    'name': 'Jack',
    'age': 22,
    'gender': 'male'
  }
```
4. For inserting a document use the <kbd>insert_one()<kbd> method on the collection and pass a document.
```py
  Syntax: db.<collection-name>.insert_one(document)
```
```py
  >>> db.users.insert_one({
    'name': 'Kim',
    'age': 22,
    'gender': 'm'
  })
```
> The <kbd>insert_one()</kbd> method will return the <kbd>_id</kbd> of the document.
5. Mongo DB allows us to insert multiple documents using the <kbd>insert_many()</kbd> method where we pass a list of documents.
```py
  >>> db.users.insert_many([
    {
      'name': 'Jessy',
      'age': 25,
      'gender': 'f'
    }, 
    {
      'name': 'Harry',
      'age': 21,
      'gender': 'm',
      'phone': '3423432'
    },
    {
      'name': 'Parker',
      'age': 20
    }
  ])
```
> Notice in the above example that the number of attributes in each document is not the same.<br>
> Mongo DB allows documents in a collection to have varying number of fields.

### Selecting documents
1. We use <kbd>find()</kbd> method on the collection to retrieve all the documents in a collection.
1. The <kbd>find()</kbd> returns a <kbd>cursor</kbd> object which we can iterate over to get each document.
```py
  >>> results = db.users.find()
  >>> for doc in results:
  ...   print(doc)
```
```py
  Output:
  { "_id" : ObjectId("6225d56bcd8ac96f2e3f2b2a"), "name" : "Kim", "age" : 22, "gender" : "m" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2b"), "name" : "Jessy", "age" : 25, "gender" : "f" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2c"), "name" : "Harry", "age" : 21, "gender" : "m", "phone" : "3423432" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2d"), "name" : "Parker", "age" : 20 }
```

### Selecting documents with filters
1. Inorder to select documents based on filters pass the filter conditions to the <kbd>find()</kbd> method.
1. Lets get the documents whose name is 'Kim'.
```py
  >>> results = db.users.find({'name': 'Kim'})
  >>> for doc in results:
  ...   print(doc)
```
3. Running the above code gives us the output:
```py
  { "_id" : ObjectId("6225d56bcd8ac96f2e3f2b2a"), "name" : "Kim", "age" : 22, "gender" : "m" }
```
> The <kbd>find()</kbd> returns all of the documents which match the conditions. If you only want the first matching document, then use the <kbd>find_one()</kbd> method.
4. Use filters like <kbd>$gt</kbd>, <kbd>$lt</kbd>, <kbd>eq</kbd>, <kbd>$gte</kbd> for filtering values based on whether the value is `greater` or `lesser` or `equal to` a given value.

### Projecting specific fields in a collection
1. Inorder to select specific attributes in a collection, pass a json object containing the attributes name as key and `1` as value to the <kbd>find()</kbd> method as a second argument.
1. Lets only select the name fields of all of the documents in `users` collection.
```py
  >>> results = db.users.find({}, {'name': 1})
  >>> for doc in results:
  ...   print(doc)
```
```py
  Output:
  { "_id" : ObjectId("6225d56bcd8ac96f2e3f2b2a"), "name" : "Kim" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2b"), "name" : "Jessy" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2c"), "name" : "Harry" }
  { "_id" : ObjectId("6225d56ccd8ac96f2e3f2b2d"), "name" : "Parker" }
```
> The `_id` field is returned always, to avoid this use `{'name': 1, '_id': 0}` in <kbd>find()</kbd> method.

### Updating documents
1. Use the <kbd>update_one()</kbd> method to update a document in a collection.
1. The <kbd>update_one()</kbd> method takes two parameters. The first is the filter to find the document and the second is the updation values of the document.
1. Syntax: <kbd>db.\<collection-name>.update_one(\<filter>, \<update-rule>)
1. The update rule is a json object with <kbd>$set</kbd> as key and the values as an object.
1. Lets update the age of the user `Kim` to 25.
```py
  >>> db.users.update_one(
        {'name': 'Kim'}, 
        {'$set': {'age': 25}}
      )
```
6. The above query updates the age of the user Kim to 25.
> - Update one only updates the first matching record.<br>
> - $set value is a object which can update more than 1 field of the document and can also be used to add a new field to the document.
7. For updating multiple records use the <kbd>update_many()</kbd> method which updates all the documents which matches the conditions.

### Deleting documents
1. Deleting documents is similar to selecting documents where documents which match the condition are deleted.
1. Use the <kbd>delete_one()</kbd> method to delete the first matching record.
1. Syntax: <kbd>db.\<collection-name>.delete_one(\<filter-rule>)
1. Lets delete the user whose name is `Kim`.
```py
  >>> db.users.delete_one({'name': 'Kim'})
```
> - The above statement deletes the user document whose name is Kim.
> - This is a simple filter. However, you can use all complex filter involving <kbd>$gt</kbd>, <kbd>$lt</kbd>, <kbd>eq</kbd>, etc.
5. Similarly to delete more than one matching document, use the <kbd>delete_many()</kbd> method.
6. Lets delete all of the users whose age is less than 25.
```py
  >>> db.users.delete_many({'age': {'$lt': 25}})
```

- Hope that this cheat sheet gave you a quick overview on CRUD operations in Mongo DB from a Flask app (although we mainly focused on python rather than Flask)
- Incase if you liked it, Give it a ⭐.

## Maintainer:
<a href="https://github.com/ASHIK11ab">
  <img style="border-radius: 50px" src="https://avatars2.githubusercontent.com/u/58099865?s=460&u=dc835e2281a9265edf2b48059f1c8151be89a1b1&v=4" width="70px" height = "70px"> 
</a> 

[Ashik Meeran Mohideen](https://github.com/ASHIK11ab)

&copy; copyrights 2022. All rights reserved.

Licensed under [MIT LICENSE](https://github.com/ASHIK11ab/cheatsheets/blob/main/LICENSE)