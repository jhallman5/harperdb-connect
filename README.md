# HarperDB Connect 

[![Build Status](https://travis-ci.org/Ethan-Arrowood/harperdb-connect.svg?branch=master)](https://travis-ci.org/Ethan-Arrowood/harperdb-connect)
[![Coverage Status](https://coveralls.io/repos/github/Ethan-Arrowood/harperdb-connect/badge.svg?branch=master)](https://coveralls.io/github/Ethan-Arrowood/harperdb-connect?branch=master)

[![https://nodei.co/npm/harperdb-connect.png?downloads=true&downloadRank=true&stars=true](https://nodei.co/npm/harperdb-connect.png?downloads=true&downloadRank=true&stars=true)](https://www.npmjs.com/package/harperdb-connect)

Developed by Ethan Arrowood

- [API Reference](#API_REFERENCE)
- Contributing (Coming Soon)

This API is to be used with HarperDB Community Edition

<a href="API_REFERENCE"></a>

- [HarperDBConnect](#harperdbconnect)
- [setAuthorization](#setauthorization)
- [setDefaultOptions](#setdefaultoptions)
- [connect](#connect)
- [request](#request)

# API Reference

## module.exports

<a href="harperdbconnect"></a>

### _.HarperDBConnect\(\[username\], \[password\], \[url\]\)_

#### Parameters:

* username \(String\): Nonempty database authorization username string. _Optional_
* password \(String\): Nonempty database authorization password string. _Optional_
* url \(String\): 
  * Nonempty database endpoint. Make sure to include port number. `username` and `password` must be defined if `url` is defined. _Optional_
  * By providing the url, the constructor then returns a `Promise` . If succesfully connected it resolves with a reference to the new HarperDBConnect instance; otherwise, it rejects the `Error` string from [connect](#connect).

#### Example:

Instantiate HarperDBConnect with username and password:

```js
const db = new HarperDBConnect('admin', 'Password123!')
```

Instantiate HarperDBConnect with username, password, and url. Remember that by providing the url, the constructor returns a `Promise` instead:

```js
const connectToDB = async () => {
    let db;
    try {
        db = await new HarperDBConnect('admin', 'Password123!', 'http://localhost:5000')
    } catch (err) {
        console.log(`Error: ${err}`)
    }
}
```

#### Discussion:

`HarperDBConnect` has some internal properties that are defined during instantiation. `isConnected` is instantiated `false` and then toggled `true` when the database succesfully connects to the database endpoint via the [connect](#connect) method. `authorization` is a Base64 encoded string generated by combining the `username` and `password` in the [setAuthorization](#setauthorization) method. `options` is instantiated as an empty object; if `url` is provided, `options.url` is set if the connection is verified. 

## HarperDBConnect

<a href="setauthorization"></a>

### _.setAuthorization\(username, password\)_ 
Returns the generated auth string and sets the auth to the options object under `headers.authorization`.
#### Parameters:
* username \(String\): Nonempty database authorization username string. _Optional_
* password \(String\): Nonempty database authorization password string. _Optional_
#### Example:
```js
const authToken = db.setAuthorization('admin', 'Password123!')
```
#### Discussion:
This method is used internally by the constructor. You may also call it if for some reason your authorization settings change. This does not make a call to `connect` so if you do use this method make sure to call connect afterwards.

<a href="setdefaultoptions"></a>

### _.setDefaultOptions\(\[options\]\)_ 
Set your default options for the `request` method to use. This is useful for settings such as `method`, `json`, `headers.content-type`. 
#### Parameters:
- options \(Object\): options to be merged and set as default
#### Example:
```js
db.setDefaultOptions({
  method: 'POST',
  headers: {
    'content-type': 'application/json'
  },
  json: true
})
```

<a href="connect"></a>

### _.connect\(\[url\]\)_ 
Attempts to connect to the HarperDB Database. Returns a promise and will set the url on the default options if successful.
#### Parameters:
- url \(String\): string denoting URL at which HarperDB instance is on.
#### Example:
```js
db.connect('http://localhost:5000')
  .then(() => console.log('Connected!'))
  .catch(err => console.log(err))
```
#### Discussion:
This method is used internally if you attempt to connect via the constructor. 

<a href="request"></a>

### _.request\(\[queryOrOptions\], \[useDefault\]\)_ 
Use this method to make requests against the HarperDB instance. It will use the default options by default. Returns a promise resolving the response from the database. 

#### Parameters:
- queryOrOptions \(Object\): if you are using the default options you only need to pass the database query; otherwise, you must pass the entire request object and put your query in the `body` property.
- useDefault \(boolean\): Default true. If false you must pass options object as first parameter.
#### Example:
Using defaults
```js
db.request({ operation: "describe_all" })
  .then(res => console.log(res))
  .catch(err => console.error(err))
```
Without defaults
```js
const db = new HarperDBConnect('admin', 'Password123!', 'http://localhost:5000')
db.request({
    method: 'POST',
    url: db.options.url,
    headers: {
      'content-type': 'application/json',
      authorization: db.authorization,
    },
    json: true,
    body: { operation: 'describe_all' },
  })
  .then(res => console.log(res))
  .catch(err => console.error(err))
```
#### Discussion:
It is recommended you use the default options. 