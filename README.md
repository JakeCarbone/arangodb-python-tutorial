# ArangoDB Tutorial

This is a tutorial to using Python in conjunction with ArangoDB. It loosely follows this [tutorial from ArangoDB](https://www.arangodb.com/tutorials/tutorial-python/)

## Installation

To install ArangoDB on your setup choose your option from [this site](https://www.arangodb.com/download-major/) if you are using windows you can pick the first option under the NSIS packages section. Follow the installation and when it asks you to pick a password you can just leave it blank.

You can install the python client with this code `pip install pyarango --user`.

You should be able to open the arango interface by going to this your [local host port 8529](http://127.0.0.1:8529). Here you can login with the user name `root` and by leaving the password blank (if that is what you did at setup).

## Connecting to the database

In order to operate on ArangoDB servers and databases from within your application, you need to establish a connection to the server then use it to open or create a database on that server.

PyArango manages server connections through the conveniently named Connection class.


```python
from pyArango.connection import *
conn = Connection(username="root", password="")
```

When this code executes, it initializes the server connection on the `conn` variable. By default, pyArango attempts to establish a connection to http://127.0.0.1:8529. That is, it wants to initialize a remote connection to your local host on port 8529. If you are hosting ArangoDB on a different server or use a different port, you need to set these options when you instantiate the `Connection` class.

## Creating and Opening Databases

With a connection to the ArangoDB Server, you can open or create a database on the server and begin to operate on it. The `createDatabase()` method on the server connection handles both operations, returning a `Database` instance. Now go to the [ArangoDB database UI](http://127.0.0.1:8529) and you should be able to log into your new database.


```python
db = conn.createDatabase(name="school")
```


When the `school` database does not exist, pyArango creates it on the server connection. When it does exist, it attempts to open the database. You can also open an existing database by using its name as a key on the server connection. For instance:


```python
db = conn["school"]
db
```

`Output:`


    ArangoDB database: school



## Creating Collections

ArangoDB groups documents and edges into collections. This is similar to the concept of tables in Relational databases, but with the key difference that collections are schema-less.

In pyArango, you can create a collection by calling the `createCollection()` method on a given database. For instance, in the above section you created a `school` database. You might want a collection on that database for students.


```python
studentsCollection = db.createCollection(name="Students")
db["Students"]
```

`Output:`


    ArangoDB collection name: Students, id: 1181, type: document, status: loaded



## Creating Documents

With the database and collection set up, you can begin adding data to them. Continuing the comparison to Relational databases, where a collection is a table, a document is a row on that table. Unlike rows, however, documents are schema-less. You can include any arrangement of values you need for your application.

For instance, add a student to the collection:


```python
doc1 = studentsCollection.createDocument()
doc1["name"] = "John Smith"
doc1
```

`Output:`


    ArangoDoc '_id: None, _key: None, _rev: None': <store: {'name': 'John Smith'}>




```python
doc2 = studentsCollection.createDocument()
doc2["firstname"] = "Emily"
doc2["lastname"] = "Bronte"
doc2
```

`Output:`


    ArangoDoc '_id: None, _key: None, _rev: None': <store: {'firstname': 'Emily', 'lastname': 'Bronte'}>



The document shows its `_id` as “None” because you haven’t yet saved it to ArangoDB. This means the variable exists in your Python code, but not the database. ArangoDB constructs `_id` values by pairing the collection name with the `_key` value. It also handles the assignment for you, you just need to set the key and save the document.


```python
doc1._key = "johnsmith"
doc1.save()
```

Rather than enter and save the data for all the students manually, you might want to enter the data through a loop rather than individual calls. For instance:


```python
students = [('Oscar', 'Wilde', 3.5), ('Thomas', 'Hobbes', 3.2), 
('Mark', 'Twain', 3.0), ('Kate', 'Chopin', 3.8), ('Fyodor', 'Dostoevsky', 3.1), 
('Jane', 'Austen',3.4), ('Mary', 'Wollstonecraft', 3.7), ('Percy', 'Shelley', 3.5), 
('William', 'Faulkner', 3.8), ('Charlotte', 'Bronte', 3.0)]

for (first, last, gpa) in students:
    doc = studentsCollection.createDocument()
    doc['name'] = "%s %s" % (first, last)
    doc['gpa'] = gpa 
    doc['year'] = 2017
    doc._key = ''.join([first, last]).lower() 
    doc.save()
```

Now if you go to your [Students collection](http://127.0.0.1:8529/_db/school/_admin/aardvark/index.html#collection/Students/documents/1) you should see it populated with data.

## Reading Documents

Eventually, you’ll need to access documents in the database. The easiest way to do this is with the `_key` value.

For instance, the school database now has several students. Imagine it as part of a larger application with more data available on each student and you would like to check the GPA of a particular student:


```python
def report_gpa(document):
    print("Student: %s" % document['name'])
    print("GPA:     %s" % document['gpa'])
```


```python
kate = studentsCollection['katechopin']
report_gpa(kate)
```

`Output:`

    Student: Kate Chopin
    GPA:     3.8

### Updating Documents

When you read a document from ArangoDB into your application, you  create a local copy of the document. You can then operate on the  document, making whatever changes you like to it, then push the results  to the database using the `save()` method.

For instance, each semester as the final grades come in from their  classes, you need to update the students’ grade point averages on the  database. Given that this happens frequently, you might want to create a specific function to handle the update:

```python
def update_gpa(key, new_gpa):
    doc = studentsCollection[key]
    doc['gpa'] = new_gpa
    doc.save()
```

## Listing Documents

Occasionally, you may want to operate on all documents in a given collection. Using the fetchAll() method, you can retrieve and iterate over a list of documents. For instance, say it’s the end of the semester and you want to know which students have a grade point average above 3.5:


```python
def top_scores(col, gpa):
    print("Top Soring Students:")
    for student in col.fetchAll():
        try:
            if student['gpa'] >= gpa:
                print("- %s" % student['name'])
        except:
            pass

top_scores(studentsCollection, 3.5)
```

`Output:`

    Top Soring Students:
    - Oscar Wilde
    - Kate Chopin
    - Mary Wollstonecraft
    - Percy Shelley
    - William Faulkner


## Removing Documents

Eventually, you may want to remove documents from the database. This can be accomplished with the `delete()` method. For instance, say that the student Thomas Hobbes has decided to move to another city:


```python
tom = studentsCollection["thomashobbes"]
tom.delete()
```

Now because Thomas has been deleted, you should get an error if you ran the following code.


```python
studentsCollection["thomashobbes"]
```

`Output:`


    DocumentNotFoundError: Unable to find document with _key: thomashobbes. Errors: {'code': 404, 'error': True, 'errorMessage': 'document not found', 'errorNum': 1202}


## AQL Usage

In addition to the Python methods shown above, ArangoDB also provides a query language, (called AQL), for retrieving and modifying documents on the database. In pyArango, you can issue these queries using the `AQLQuery()` method.

For instance, say you want to retrieve the keys for all documents in ArangoDB:


```python
aql = "FOR x IN Students RETURN x._key"
queryResult = db.AQLQuery(aql, rawResults=True, batchSize=100)
for key in queryResult:
    print(key)
```

`Output:`

    charlottebronte
    fyodordostoevsky
    janeausten
    katechopin
    marktwain
    marywollstonecraft
    oscarwilde
    percyshelley
    williamfaulkner


In the above example, the `AQLQuery()` method takes the AQL query as an argument, with two additional options:

* `rawResults` Defines whether you want the actual results returned by the query.
* `batchSize` When the query returns more results than the given value, the pyArango driver automatically asks for new batches.

Bear in mind, the order of the documents isn’t guaranteed. In the event that you need the results in a particular order, add a sort clause to the AQL query. There is more at the bottom of the [tutorial](https://www.arangodb.com/tutorials/tutorial-python/) about running AQL scripts from the Python interface.

The end :)
