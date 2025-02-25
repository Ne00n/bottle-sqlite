=====================
Bottle-SQLite
=====================

.. image:: https://travis-ci.org/bottlepy/bottle-sqlite.png?branch=master
    :target: https://travis-ci.org/bottlepy/bottle-sqlite
    :alt: Build Status - Travis CI

SQLite is a self-contained SQL database engine that runs locally and does not 
require any additional server software or setup. The sqlite3 module is part of the 
Python standard library and already installed on most systems. It it very useful 
for prototyping database-driven applications that are later ported to larger 
databases such as PostgreSQL or MySQL. 

This plugin simplifies the use of sqlite databases in your Bottle applications. 
Once installed, all you have to do is to add a ``db`` keyword argument 
(configurable) to route callbacks that need a database connection.

Installation
===============

Install with the following commands::

    $ git clone git://github.com/Ne00n/bottle-sqlite.git
    $ cd bottle-sqlite
    $ python setup.py install

Usage
===============

Once installed to an application, the plugin passes an open 
:class:`sqlite3.Connection` instance to all routes that require a ``db`` keyword 
argument::

    import bottle

    app = bottle.Bottle()
    plugin = bottle.ext.sqlite.Plugin(dbfile='/tmp/test.db')
    app.install(plugin)

    @app.route('/show/:item')
    def show(item, db):
        row = db.execute('SELECT * from items where name=?', item).fetchone()
        if row:
            return template('showitem', page=row)
        return HTTPError(404, "Page not found")

Routes that do not expect a ``db`` keyword argument are not affected.

The connection handle is configured so that :class:`sqlite3.Row` objects can be 
accessed both by index (like tuples) and case-insensitively by name. At the end of 
the request cycle, outstanding transactions are committed and the connection is 
closed automatically. If an error occurs, any changes to the database since the 
last commit are rolled back to keep the database in a consistent state.

Configuration
=============

The following configuration options exist for the plugin class:

* **dbfile**: Database filename (default: in-memory database).
* **keyword**: The keyword argument name that triggers the plugin (default: 'db').
* **autocommit**: Whether or not to commit outstanding transactions at the end of the request cycle (default: True).
* **dictrows**: Whether or not to support dict-like access to row objects (default: True).
* **text_factory**: The text_factory for the connection (default: unicode).
* **functions**: Add user-defined functions for use in SQL, should be a dict like ``{'name': (num_params, func)}`` (default: None).
* **aggregates**: Add user-defined aggregate functions, should be a dict like ``{'name': (num_params, aggregate_class)}`` (default: None).
* **collations**: Add user-defined collations, should be a dict like ``{'name': callable}`` (default: None).
* **extensions**: Load extensions on connect. Should be a list of extension names. (default: None).

You can override each of these values on a per-route basis:: 

    @app.route('/cache/:item', sqlite={'dbfile': ':memory:'})
    def cache(item, db):
        ...
   
or install two plugins with different ``keyword`` settings to the same application::

    app = bottle.Bottle()
    test_db = bottle.ext.sqlite.Plugin(dbfile='/tmp/test.db')
    cache_db = bottle.ext.sqlite.Plugin(dbfile=':memory:', keyword='cache')
    app.install(test_db)
    app.install(cache_db)

    @app.route('/show/:item')
    def show(item, db):
        ...

    @app.route('/cache/:item')
    def cache(item, cache):
        ...


Changelog
=========

* **0.2** 2020-10-03
    * Fixed ``text_factory`` parameter.
    * Added ``functions``, ``aggregates``, ``collations`` and ``extensions`` parameters.
    * Stopped testing for dead Python versions.
