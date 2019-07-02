# node-red-contrib-firebase-admin

A node-red module that wraps the server-side  admin SDK of firebase, firestore, et.c.

# Overview
The main difference of this module and all other firebase/store modules for node-red is
that it takes a service account credential token as configuration.

This means that the nodes can run outside of the normal security rules, in admin mode, which is useful when running on the back-end.

The configuration needs two parameters, the first is the text of the service account credentials private key file. This can be found by going to the firebase console, then to the cogwheel icon at the top left then to project settings and then "Service Accounts". Click "Generate new private key".
The second parameter is the database url for the database, and the url has the format of "https://myprojectname.firebaseio.com" and can also be found in the above place in the firebase console.

After initializing, the reference to the firebase SDK is stored in the global context variable 'firebase', which can be used
in any function like this;

    let fb = global.get('firebase')
    fb.firestore().doc('foo/bar').get().then((ref)=>{
        let d = ref.data()
        node.send( {payload: {data: d}});
    })

# Realtime Database (rtdb) Nodes

## rtdb-get
Get data from a path in the rtdb database

input: {"payload": {"path": "foo/bar"}}

output: <whatever data was at the path "foo/bar" in the rtdb database>

## rtdb-set
Set data at a path in the rtdb database. Use "on" snapshot so will fire every time the data at the path changes and so drive flow execution from that point.

## rtdb-push
Pushes the new object onto an array under the path

## rtdb-query
Set up a reactive query for a path in the rtdb database. 

input: {"payload": {"path": "foo/bar", queries:[], on: "value}}

on: "value" (can also be "child_added", "child_removed", "child_changed", "child_moved"). 
If an "on" property is missing, on: "value" is assumed as default

Where each query is an object that can look like either of the following examples;
    
- {"startAt": "foo"}
- {"endAt": "bar"}
- {"equalTo": "quux"}
- {"orderBy": "child", "value": "height"}  (can also be "key" or "value)
- {"limitTo": "last", "value": 3}  (can also be "first")

output: [an array of results for the query]


# Firestore nodes

## firestore-get
Get data from a document path in the firestore database

input: {"payload": {"path": "foo/bar"}}

output: <the document at the path "foo/bar" in the firestore database>

## firestore-set
Set data at a path in the firestore database. Uses "onSnapshot" so will fire every time the data at the path changes and so drive flow execution from that point.

input: {"payload": {"path": "foo/bar"}, {"some": object, "foo": 17}}

## firestore-add
Adds the new object under the collection the path describes and assigns it a random id

input: {"payload": {"path": "foo/bar"}, {"some": object, "foo": 17}}

output: The id of the new document

## firestore-query
Set up a reactive query for a collection in the firestore database.

input: 

    {
        "payload": {
            "path": "foo/bar", 
            "queries":[], 
            "limit": 7, 
            "startAt": 4000,  // follows the orderBy property
            "endAt": 4050, // follows the orderBy property
            "orderBy": "shoeSize",
            "orderDirection": "asc", //default is "desc"
            "queries":[
                {"company", "==", "ACME"},
                {"createdAt", ">", 1560099394242}
            ]
        }

output: An array of the results of the query.


# Storage nodes

## storage-read
Read file data from a file at a given path under a given cloud storage bucket. The default bucket to be used can be set in the general firebase SDK settings.
If the payload defines an optional bucket property, it will override the default bucket settings.

input: {"payload":{"bucket": "xyzzyz123.appspot.com", "path": "myFile.txt"}}

npm publish .output: Buffer object containing the binary file contents. Can easily be converted to a string by calling toString() on the Buffer.
 
## storage-write
Writes the content of  JavaScript Buffer object to a file path in a storage bucket. 

input: 

    {
        "payload": { 
            "bucket": "abc.appspot.com", // optional, is otherwise set as node config
            "path": "foo/bar/baz.json", // optional, see above
            "contents": <Buffer obj>,
            "contentType": "application/json" }, // optional
            "metadata": { "very":"interesting"}, // optional
            "public": true, // optional
            "private": false // optional
        }
    } 
    

# Auth nodes

TBD

