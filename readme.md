# Node Cellar Sample Application with Backbone.js, Twitter Bootstrap, Node.js, Express, and MongoDB #

"Node Cellar" is a sample CRUD application built with with Backbone.js, Twitter Bootstrap, Node.js,
Express, and MongoDB by [Christophe Coenraets](http://coenraets.org/blog/).

The application allows you to browse through a list of wines, as well as add, update, and delete wines.

This version of "Node Cellar" has been modified to use [thin-orm](https://github.com/on-point/thin-orm)
and [sqlite](http://www.sqlite.org) instead of MongoDB.

The process of converting from MongoDB to thin-orm is fairly simple. Throughout the application, change
all references to "_id" back to the more conventional "id."

Then in the database access layout, swap the MongoDB API for the thin-orm API. Here are some examples.

### Initial configuration and population of the database

MongoDB:
```js
db.collection('wines', function(err, collection) {
    collection.insert(wines, {safe:true}, function(err, result) {});
});
```

Thin-orm:
```js
db.run("create table wines (id INTEGER PRIMARY KEY, "
     + "                    name VARCHAR(255), "
     + "                    year INTEGER, "
     + "                    grapes VARCHAR(255), "
     + "                    country VARCHAR(255), "
     + "                    region VARCHAR(255), "
     + "                    description VARCHAR(65535), "
     + "                    picture VARCHAR(255))",
    function(err, result) {
        if (err) {
            console.log(err);
            return;
        }
        async.forEachSeries(wines, function(wine, callback) {
            winesClient.create({ data: wine }, callback);
        });
    }
);
```

### Fetching a record

MongoDB:
```js
exports.findById = function(req, res) {
    var id = req.params.id;
    console.log('Retrieving wine: ' + id);
    db.collection('wines', function(err, collection) {
        collection.findOne({'_id':new BSON.ObjectID(id)}, function(err, item) {
            res.send(item);
        });
    });
};
```

Thin-orm:
```js
exports.findById = function(req, res) {
    console.log('Retrieving wine: ' + req.params.id);

    winesClient.findById(req.params.id, res);
};
```

### Adding a record

MongoDB:
```js
exports.addWine = function(req, res) {
    var wine = req.body;
    console.log('Adding wine: ' + JSON.stringify(wine));
    db.collection('wines', function(err, collection) {
        collection.insert(wine, {safe:true}, function(err, result) {
            if (err) {
                res.send({'error':'An error has occurred'});
            } else {
                console.log('Success: ' + JSON.stringify(result[0]));
                res.send(result[0]);
            }
        });
    });
}
```

Thin-orm:
```js
exports.addWine = function(req, res) {
    var wine = req.body;
    console.log('Adding wine: ' + JSON.stringify(wine));

    winesClient.create({ data: wine }, res);
};
```
### Updating a record

MongoDB:
```js
exports.updateWine = function(req, res) {
    var id = req.params.id;
    var wine = req.body;
    delete wine._id;
    console.log('Updating wine: ' + id);
    console.log(JSON.stringify(wine));
    db.collection('wines', function(err, collection) {
        collection.update({'_id':new BSON.ObjectID(id)}, wine, {safe:true}, function(err, result) {
            if (err) {
                console.log('Error updating wine: ' + err);
                res.send({'error':'An error has occurred'});
            } else {
                console.log('' + result + ' document(s) updated');
                res.send(wine);
            }
        });
    });
}
```

Thin-orm:
```js
exports.updateWine = function(req, res) {
    var id = req.params.id;
    var wine = req.body;
    console.log('Updating wine: ' + id);
    console.log(JSON.stringify(wine));

    winesClient.update({ criteria: { id: id }, data: wine}, res);
};
```

### Deleting a record

MongoDB:
```js
exports.deleteWine = function(req, res) {
    var id = req.params.id;
    console.log('Deleting wine: ' + id);
    db.collection('wines', function(err, collection) {
        collection.remove({'_id':new BSON.ObjectID(id)}, {safe:true}, function(err, result) {
            if (err) {
                res.send({'error':'An error has occurred - ' + err});
            } else {
                console.log('' + result + ' document(s) deleted');
                res.send(req.body);
            }
        });
    });
}
```

Thin-orm:
```js
exports.deleteWine = function(req, res) {
    var id = req.params.id;
    console.log('Deleting wine: ' + id);

    winesClient.remove(id, res);
};
```