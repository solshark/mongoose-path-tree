## mongoose-path-tree
[![Build Status](https://travis-ci.org/swayf/mongoose-path-tree.png)](https://travis-ci.org/swayf/mongoose-path-tree)

Implements the materialized path strategy with cascade child re-parenting on delete for storing a hierarchy of documents with mongoose
Version with all collected features and fixes from mongoose-tree, mongoose-tree-fix, mongoose-tree2, mongoose-reparenting-tree

# Usage

Install via NPM

    $ npm install mongoose-path-tree

## Options

```javascript
Model.plugin(tree, {
  pathSeparator : '#'              // Default path separator
  onDelete :      'REPARENT'       // Can be set to 'DELETE' or 'REPARENT'. Default: 'REPARENT'
  numWorkers:     5                // Number of stream workers
  idType:         Schema.ObjectId  // Type used for _id. Can be, for example, String generated by shortid module
})
```

Then you can use the plugin on your schemas

```javascript
var tree = require('mongoose-tree');

var UserSchema = new Schema({
  name : String
});
UserSchema.plugin(tree);
var User = mongoose.model('User', UserSchema);

var adam = new User({ name : 'Adam' });
var bob = new User({ name : 'Bob' });
var carol = new User({ name : 'Carol' });

// Set the parent relationships
bob.parent = adam;
carol.parent = bob;

adam.save(function() {
  bob.save(function() {
    carol.save();
  });
});
```

At this point in mongoDB you will have documents similar to

    {
      "_id" : ObjectId("50136e40c78c4b9403000001"),
      "name" : "Adam",
      "path" : "50136e40c78c4b9403000001"
    }
    {
      "_id" : ObjectId("50136e40c78c4b9403000002"),
      "name" : "Bob",
      "parent" : ObjectId("50136e40c78c4b9403000001"),
      "path" : "50136e40c78c4b9403000001#50136e40c78c4b9403000002"
    }
    {
      "_id" : ObjectId("50136e40c78c4b9403000003"),
      "name" : "Carol",
      "parent" : ObjectId("50136e40c78c4b9403000002"),
      "path" : "50136e40c78c4b9403000001#50136e40c78c4b9403000002#50136e40c78c4b9403000003"
    }

The path is used for recursive methods and is kept up to date by the plugin if the parent is changed

# API

### getChildren

Signature:

    getChildren([filters], [fields], [options], [recursive], cb);

args are additional filters if needed.
if recursive is supplied and true, subchildren are returned

Based on the above hierarchy:

```javascript
adam.getChildren(function(err, users) {
  // users is an array of with the bob document
});

adam.getChildren(true, function(err, users) {
  // users is an array with both bob and carol documents
});
```

### getChildrenTree

Signature as method:

    getChildrenTree([args], cb);

Signature as static:

    getChildrenTree([rootDoc], [args], cb);

return a recursive tree of sub-children.

args is an object you can defined with theses properties :

    filters: mongoose query filter, optional, default null
      example: filters: {owner:myId}

    fields: mongoose fields, optional, default null (all fields)
      example: fields: "_id name owner"

    options: mongoose query option, optional, default null
      example: options:{{sort:'-name'}}

    minLevel: level at which will start the search, default 1
      example: minLevel:2

    recursive: boolean, default true
      make the search recursive or only fetch children for the specified level
      example: recursive:false

    allowEmptyChildren: boolean, default true
      if true, every child not having children will have 'children' attribute (empty array)
      if false, every child not having children will not have 'children' attribute

    Example :

```javascript
var args = {
  filters: {owner:myId},
  fields: "_id name owner",
  minLevel:2,
  recursive:true,
  allowEmptyChildren:false
}

getChildrenTree(args,myCallback);
```

Based on the above hierarchy:

```javascript
adam.getChildrenTree( function(err, users) {

    /* if you dump users, you will have something like this :
    {
      "_id" : ObjectId("50136e40c78c4b9403000001"),
      "name" : "Adam",
      "path" : "50136e40c78c4b9403000001"
      "children" : [{
          "_id" : ObjectId("50136e40c78c4b9403000002"),
          "name" : "Bob",
          "parent" : ObjectId("50136e40c78c4b9403000001"),
          "path" : "50136e40c78c4b9403000001#50136e40c78c4b9403000002"
          "children" : [{
              "_id" : ObjectId("50136e40c78c4b9403000003"),
              "name" : "Carol",
              "parent" : ObjectId("50136e40c78c4b9403000002"),
              "path" : "50136e40c78c4b9403000001#50136e40c78c4b9403000002#50136e40c78c4b9403000003"
          }]
      }]
    }
    */

});

```

### getAncestors

Signature:

    getAncestors([filters], [fields], [options], cb);

Based on the above hierarchy:

```javascript
carol.getAncestors(function(err, users) {
  // users is an array of adam and bob
})
```

### level

Equal to the level of the hierarchy

```javascript
carol.level; // equals 3
```

# Tests

To run the tests install mocha

    npm install mocha -g

and then run

    mocha


