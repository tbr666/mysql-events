# Update:

I am not the owner of this code and the content has been forked from the original repository
https://github.com/spencerlambert/mysql-events

In my fork the mysql-events initialization method has been changed to support custom error handling in the file where this module is used.
The previous way of handling did not work for some specific situations like recovering when the database is restarted. The new connection was set up and was active on node process level, but all connection objects of the event were indicated as disconnected and log updates or event triggers were not detected. Sometimes a developer should decide just to create completely new MySQLEvents object and therefore the custom event handling should be implemented for more flexibility.

I haven't been activly updating this code.  Others have been activley making updates.  Please also checkout their work.

[https://github.com/rodrigogs/mysql-events](https://github.com/rodrigogs/mysql-events)

Note: This version doesn't follow my documentation, so please reference the supplied example code for setup.

[https://github.com/rodrigogs/mysql-events/blob/master/examples/watchWholeInstance.js](https://github.com/rodrigogs/mysql-events/blob/master/examples/watchWholeInstance.js)

# mysql-events
A Node JS NPM package that watches a MySQL database and runs callbacks on matched events.

This package is based on the [ZongJi](https://github.com/nevill/zongji) node module. Please make sure that you meet the requirements described at [ZongJi](https://github.com/nevill/zongji), like MySQL binlog etc.

# Quick Start
```javascript
var MySQLEvents = require('mysql-events');
var dsn = {
  host:     _dbhostname_,
  user:     _dbusername_,
  password: _dbpassword_,
};
var mysqlEventWatcher = MySQLEvents(dsn);
var watcher =mysqlEventWatcher.add(
  'myDB.table.field.value',
  function (oldRow, newRow, event) {
     //row inserted
    if (oldRow === null) {
      //insert code goes here
    }

     //row deleted
    if (newRow === null) {
      //delete code goes here
    }

     //row updated
    if (oldRow !== null && newRow !== null) {
      //update code goes here
    }

    //detailed event information
    //console.log(event)
  }, 
  'match this string or regex'
);
```

# Installation
```sh
npm install mysql-events
```

# Usage
- Import the module into your application
```javascript
var MySQLEvents = require('mysql-events');
```

- Instantiate and create a database connection
```sh
var dsn = {
  host:     'localhost',
  user:     'username',
  password: 'password'
};
var myCon = MySQLEvents(dsn);
```

Make sure the database user has the privilege to read the binlog on database that you want to watch on.

- Use the returned object to add new watchers
```sh
var event1 = myCon.add(
  'dbName.tableName.fieldName.value',
  function (oldRow, newRow, event) {
    //code goes here
  }, 
  'Active'
);
```

This will listen to any change in the _fieldName_ and if the changed value is equal to __Active__, then triggers the callback. Passing it 2 arguments. Argument value depends on the event.

- Insert: oldRow = null, newRow = rowObject
- Update: oldRow = rowObject, newRow = rowObject
- Delete: oldRow = rowObject, newRow = null

### `rowObject`
It has the following structure:

```
{
  database: dbName,
  table: tableName,
  affectedColumns: {
    [{
      name:     fieldName1,
      charset:  String,
      type:     Number
      metedata: String
    },{
      name:     fieldName2,
      charset:  String,
      type:     Number
      metedata: String
    }]
},{
  changedColumns: [fieldName1, fieldName2],
  fields: {
   fieldName1: recordValue1,
   fieldName2: recordValue2,
     ....
     ....
     ....
   fieldNameN: recordValueN
  }
}
```

## Remove an event
```
event1.remove();
```

## Stop all events on the connection
```
myCon.stop();
```

## Additional options
In order to customize the connection options, you can provide your own settings passing a second argument object to the connection function.
```
var mysqlEventWatcher = MySQLEvents(dsn, {
  startAtEnd: false // it overrides default value "true"
});
```
You can find the list of the available options [here](https://github.com/nevill/zongji#zongji-class).

# Watcher Setup
Its basically a dot '.' seperated string. It can have the following combinations

- _database_: watches the whole database for changes (insert/update/delete). Which table and row are affected can be inspected from the oldRow & newRow
- _database.table_: watches the whole table for changes. Which rows are affected can be inspected from the oldRow & newRow
- _database.table.column_: watches for changes in the column. Which database, table & other changed columns can be inspected from the oldRow & newRow
- _database.table.column.value_: watches for changes in the column and only trigger the callback if the changed value is equal to the 3rd argument passed to the add().
- _database.table.column.regexp_: watches for changes in the column and only trigger the callback if the changed value passes a regular expression test to the 3rd argument passed to the add(). The 3rd argument must be a Javascript Regular Expression Object, like, if you want to match for a starting sting (eg: MySQL) in the value, use /MySQL/i. This will trigger the callback only if the new value starts with MySQL

# ERROR HANDLING

For error handling simply set on error handler for ZongJi instance inside MysqlEvents object.

function createMysqlEventWatcher() {

...

 mysqlEventWatcher.zongji.on('error',function(err) {
          mysqlEventWatcher.stop();
          // to avoid multiple triggering of new mysql event watcher creation
          validWatcherExists = false;
          setTimeout(createMysqlEventWatcher, 2000);
});

...

}

# LICENSE
MIT
