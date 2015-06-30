transacd
===============
[![Build Status](https://travis-ci.org/redpelicans/transacd.png?branch=transac2)](https://travis-ci.org/redpelicans/transacd)
[![Dependency Status](https://david-dm.org/redpelicans/transacd.png)](https://david-dm.org/redpelicans/transac) 
[![Coverage Status](https://coveralls.io/repos/redpelicans/transacd/badge.png?branch=transac2)](https://coveralls.io/r/redpelicans/transacd?branch=transac2)



transacd is a centralized  logging system aimed at helping monitoring business applications.

It's a very pragmatic and simple system made of a restful HTTP server based on node.js and rethinkdb, a HTML single page application written with Aurelia and clients (node.js, ruby, python, perl, ...).

Ruby, python, perl clients are not yet available, node's one is the NPM package called [transac](https://github.com/redpelicans/transac.git).

The node version is used in production, and a previous version in ruby was in use for many years in very constraints production environments.

If you have to monitor results of many processes / batches (made in different languages) and you want to identify quickly their states, and in case of failure see why they failed, this tool is for you.

It's neither Nagios nor syslog, it's more business oriented, rather than system, it's a complementary tool for applications support teams.
 

### Usage

The best way to run it is to use the [Docker image](https://github.com/redpelicans/transac-docker.git).

Without Docker do :

First setup params.js, see below, to point to your favorite rethinkdb database, then:

```javascript 
$ git clone https://github.com/redpelicans/transacd.git 
$ cd transacd
$ npm install
$ gulp serve:dist
```

Setup `params.js` like this:

```javascript 
module.exports = {
  http:{ port: 3004 },
  db: {
    host: 'rethinkdb',
    db: 'transacs',
  },
};
```
```javascript 
   $ npm run
```

That's all your server is up. Point your brower on "http://localhost:3004" an transac again and again ...
To initate transacs you need to use one the clients available (see NPM 'transac') [transac](https://github.com/redpelicans/transac.git).



### Concepts

A `transac` of course, is not a transaction, it's a set of messages associated with a name and a date.

A `transac` is uniquely defined by:
  * `name`: Name of the transac 
  * `valueDate`: default to today, but can reference usually a past date. Useful when the processing date of a process is different from the reference date of elements processed.

A `transac` can be locked: it means system will refuse to execute a new transac with same couple (`name`, `valueDate`).

A `transac` is a hierarchical structure made of `transacs`, `events` and `messages`, an `event` is made of `messages` and a `message` is mainly a timestamp `label` and a `level` of importance ['info', 'warning', 'error'].

A client can start, commit and abort a `transac`. All those actions only change `transac`'s state which can be within ['ok', 'warning', 'error'].

A `transac` can imbricate other transacs on one level: a script is launched many times a day, to avoid to pollute our journal with the same transac's name, we can define just one transac made of all the transacs generated by the script.


### HTTP API

Server is made of 4 restful requests:

* POST /transacs to register a transac
  * label: transac's label
  * valueDate: default to today
  * compound: will create a hierarchical transac, all new post with same valueDate and name will be nested in the same transac's tree.
  * locked: default false, all new post with same `valueDate` and `label` will be rejected by client API if true
  * server: optional, client's server name
  * user: optional, client's user name
  * processId: optional, client's process ID

* PUT /transacs/:id/events to add an optionnal event and messages to transac.id
  * type: ['info', 'warning', 'error', 'commit', 'abort']
  * label: `event` label
  * messages: optionnal, if not undefined, a `message` will be added to the `transac`

* GET /transacs: return JSON list of transacs
 * mode: ['value || 'processing'] to select a query between `valueDate` and `processingTime`
 * startDate, endDate: dates range for query

* GET /transacs/:id: return JSON definition of a transac (could be a simple or compound transac);

### TODO

* add other clients (python, ruby, perl, ...)
* send emails on abort or error
* integrate with other products
