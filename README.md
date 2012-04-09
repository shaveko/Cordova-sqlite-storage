Cordova/PhoneGap Native SQLitePlugin
====================================

Native interface to sqlite in a Cordova/PhoneGap plugin, working to follow the HTML5 Web SQL API as close as possible. **NOTE** that the API is now different from https://github.com/davibe/Phonegap-SQLitePlugin and is still undergoing some changes.

DISCLAIMER:

Created by @Joenoon:

Adapted to 1.5 by @coomsie

API changes and Android version by @chbrody


DISCLAIMER:

We are brand new to objective-c, so there could be problems with our code!

Installing
==========

**NOTE:** There are now 2 trees: `Cordova-iOS` for Cordova 1.5(+) and `Legacy-PhoneGap-iPhone` for PhoneGap (tested 1.3 and earlier). I am planning to add an Android version in another tree, hopefully in the near future.


PhoneGap 1.3.0
--------------

For installing with PhoneGap 1.3.0:
in PGSQLitePlugin.h file change for PhoneGaps JSONKit.h implementation.

    #ifdef PHONEGAP_FRAMEWORK
        #import <PhoneGap/PGPlugin.h>
        #import <PhoneGap/JSONKit.h>
        #import <PhoneGap/PhoneGapDelegate.h>
        #import <PhoneGap/File.h>
        #import<PhoneGap/FileTransfer.h>
    #else
        #import "PGPlugin.h"
        #import "JSON.h"
        #import "PhoneGapDelegate.h"
        #import "File.h"
    #endif

and in PGSQLitePlugin.m JSONRepresentation must be changed to JSONString:

    --- a/Plugins/PGSQLitePlugin.m
    +++ b/Plugins/PGSQLitePlugin.m
    @@ -219,7 +219,7 @@
             if (hasInsertId) {
                 [resultSet setObject:insertId forKey:@"insertId"];
             }
    -        [self respond:callback withString:[resultSet JSONRepresentation] withType:@"success"];
    +        [self respond:callback withString:[resultSet JSONString] withType:@"success"];
         }
     }

SQLite library
--------------

In the Project "Build Phases" tab, select the _first_ "Link Binary with Libraries" dropdown menu and add the library `libsqlite3.dylib` or `libsqlite3.0.dylib`.

**NOTE:** In the "Build Phases" there can be multiple "Link Binary with Libraries" dropdown menus. Please select the first one otherwise it will not work.

Native SQLite Plugin
--------------------

Drag .h and .m files into your project's Plugins folder (in xcode) -- I always
just have "Create references" as the option selected.

Take the precompiled javascript file from build/, or compile the coffeescript
file in src/ to javascript WITH the top-level function wrapper option (default).

Use the resulting javascript file in your HTML.

Look for the following to your project's Cordova.plist or PhoneGap.plist:

    <key>Plugins</key>
    <dict>
      ...
    </dict>

Insert this in there:

    <key>SQLitePlugin</key>
    <string>SQLitePlugin</string>

**NOTE:** For `Legacy-PhoneGap-iPhone` the plugin name is `PGSQLitePlugin`, no fix is expected in this project.

General Usage
=============

Android (DroidGap)
------------------

I put the information here for the sake of completeness. I have tested the DroidGap SQLitePlugin version on a simulator on PhoneGap 1.1 *ONLY* and do not guarantee what will happen in any other situation. Basically, I copied and adapted the code from storage.js and Storage.java to make a plugin version. I got the versions that were there on October 2011, so it should be OK to use them under the MIT or Apache licenses. Hereby you can take SQLitePlugin.java (it is in the wrong place but it still worked), SQLitePlugin.js, and look at my index.html, register in plugins.xml, and give it your best shot! Fork it and take it over!

Cordova iOS
-----------

**NOTE:** Please use sqlitePlugin.openDatabase() to open a database, the parameters are different from the old SQLitePlugin() constructor which is now gone. This should a closer resemblance to the HTML5/W3 SQL API.

## Coffee Script

    db = sqlitePlugin.openDatabase("my_sqlite_database.sqlite3")
    db.executeSql('DROP TABLE IF EXISTS test_table')
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)')

    db.transaction (tx) ->

      tx.executeSql "INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], (res) ->

        # success callback

        console.log "insertId: #{res.insertId} -- probably 1"
        console.log "rowsAffected: #{res.rowsAffected} -- should be 1"

        # check the count (not a part of the transaction)
        db.executeSql "select count(id) as cnt from test_table;", [], (res) ->
          console.log "rows.length: #{res.rows.length} -- should be 1"
          console.log "rows[0].cnt: #{res.rows[0].cnt} -- should be 1"

      , (e) ->

        # error callback

        console.log "ERROR: #{e.message}"

## Plain Javascript

    var db = sqlitePlugin.openDatabase("my_sqlite_database.sqlite3");

    db.executeSql('DROP TABLE IF EXISTS test_table');
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
    db.transaction(function(tx) {
      return tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(res) {
        console.log("insertId: " + res.insertId + " -- probably 1");
        console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
        return db.executeSql("select count(id) as cnt from test_table;", [], function(res) {
          console.log("rows.length: " + res.rows.length + " -- should be 1");
          return console.log("rows[0].cnt: " + res.rows[0].cnt + " -- should be 1");
        });
      }, function(e) {
        return console.log("ERROR: " + e.message);
      });
    });

## Changes in tx.executeSql() success callback

        var db = sqlitePlugin.openDatabase("my_sqlite_database.sqlite3");
        db.executeSql('DROP TABLE IF EXISTS test_table');
        db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
        db.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(res) {
                      console.log("insertId: " + res.insertId + " -- probably 1");
                      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
                      db.transaction(function(tx) {
                                     return tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
                                                          console.log("rows.length: " + res.rows.length + " -- should be 1");
                                                          return console.log("rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
                                                          });
                                     });
                      });


Legacy PhoneGap (old version)
-----------------------------

## Coffee Script

    db = new PGSQLitePlugin("my_sqlite_database.sqlite3")
    db.executeSql('DROP TABLE IF EXISTS test_table')
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)')

    db.transaction (tx) ->

      tx.executeSql "INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], (res) ->

        # success callback

        console.log "insertId: #{res.insertId} -- probably 1"
        console.log "rowsAffected: #{res.rowsAffected} -- should be 1"

        # check the count (not a part of the transaction)
        db.executeSql "select count(id) as cnt from test_table;", (res) ->
          console.log "rows.length: #{res.rows.length} -- should be 1"
          console.log "rows[0].cnt: #{res.rows[0].cnt} -- should be 1"

      , (e) ->

        # error callback

        console.log "ERROR: #{e.message}"

## Plain Javascript

    var db;
    db = new PGSQLitePlugin("my_sqlite_database.sqlite3");
    db.executeSql('DROP TABLE IF EXISTS test_table');
    db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
    db.transaction(function(tx) {
      return tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
        console.log("insertId: " + res.insertId + " -- probably 1");
        console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
        return db.executeSql("select count(id) as cnt from test_table;", [], function(res) {
          console.log("rows.length: " + res.rows.length + " -- should be 1");
          return console.log("rows[0].cnt: " + res.rows[0].cnt + " -- should be 1");
        });
      }, function(e) {
        return console.log("ERROR: " + e.message);
      });
    });

## Changes in tx.executeSql() success callback

        var db;
        db = new PGSQLitePlugin("my_sqlite_database.sqlite3");
        db.executeSql('DROP TABLE IF EXISTS test_table');
        db.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
        db.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(res) {
                      console.log("insertId: " + res.insertId + " -- probably 1");
                      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");
                      db.transaction(function(tx) {
                                     return tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
                                                          console.log("rows.length: " + res.rows.length + " -- should be 1");
                                                          return console.log("rows[0].cnt: " + res.rows.item(0).cnt + " -- should be 1");
                                                          });
                                     });
                      });


Lawnchair Adapter Usage
=======================

Include the following js files in your html:

-  lawnchair.js (you provide)
-  sqlite_plugin.js [pgsqlite_plugin.js in Legacy-PhoneGap-iPhone]
-  lawnchair_sqlite_plugin_adapter.js [lawnchair_pgsqlite_plugin_adapter.js] (must come after sqlite_plugin.js [pgsqlite_plugin.js in Legacy-PhoneGap-iPhone])



The `name` option will determine the sqlite filename. Optionally, you can change it using the `db` option.

In this example, you would be using/creating the database at: *Documents/kvstore.sqlite3* (all db's in SQLitePlugin are in the Documents folder)

    kvstore = new Lawnchair { name: "kvstore", adapter: SQLitePlugin.lawnchair_adapter }, () ->
      # do stuff

Using the `db` option you can create multiple stores in one sqlite file. (There will be one table per store.)

    recipes = new Lawnchair {db: "cookbook", name: "recipes", ...}
	ingredients = new Lawnchair {db: "cookbook", name: "ingredients", ...}


Lawnchair test
--------------

In the lawnchair-test subdirectory of Cordova-iOS or Legacy-PhoneGap-iPhone you can copy the contents of the www subdirectory into a Cordova/PhoneGap project and see the behavior of the Lawnchair test suite.

### Other notes from @Joenoon:

I played with the idea of batching responses into larger sets of
writeJavascript on a timer, however there was only a barely noticeable
performance gain.  So I took it out, not worth it.  However there is a
massive performance gain by batching on the client-side to minimize
PhoneGap.exec calls using the transaction support.