sessioncache
===============================
This is a simple library that emulates `memcache` functions using HTML5 `sessionStorage`, so that you can cache data on the client
and associate an expiration time with each piece of data. If the `sessionStorage` limit (~5MB) is exceeded, it tries to create space by removing the items that are closest to expiring anyway. If `sessionStorage` is not available at all in the browser, the library degrades by simply not caching and all cache requests return null.

Methods
-------

The library exposes 5 methods: `set()`, `get()`, `remove()`, `flush()`, and `setBucket()`.

* * *

### sessioncache.set
Stores the value in sessionStorage. Expires after specified number of minutes.
#### Arguments
1. `key` (**string**)
2. `value` (**Object|string**)
3. `time` (**number: optional**)

* * *

### sessioncache.get
Retrieves specified value from sessionStorage, if not expired.
#### Arguments
1. `key` (**string**)

#### Returns
**string | Object** : The stored value. If no value is available, null is returned.

* * *

### sessioncache.remove
Removes a value from sessionStorage.
#### Arguments
1. `key` (**string**)

* * *

### sessioncache.flush
Removes all sessioncache items from sessionStorage without affecting other data.

* * *

### sessioncache.setBucket
Appends CACHE_PREFIX so sessioncache will partition data in to different buckets
#### Arguments
1. `bucket` (**string**)

Usage
-------

The interface should be familiar to those of you who have used `memcache`, and should be easy to understand for those of you who haven't.

For example, you can store a string for 2 minutes using `sessioncache.set()`:

```js
sessioncache.set('greeting', 'Hello World!', 2);
```

You can then retrieve that string with `sessioncache.get()`:

```js
alert(sessioncache.get('greeting'));
```

You can remove that string from the cache entirely with `sessioncache.remove()`:

```js
sessioncache.remove('greeting');
```

You can remove all items from the cache entirely with `sessioncache.flush()`:

```js
sessioncache.flush();
```

You can remove only expired items from the cache entirely with `sessioncache.flushExpired()`:

```js
sessioncache.flushExpired();
```

You can also check if local storage is supported in the current browser with `sessioncache.supported()`:

```js
if (!sessioncache.supported()) {
  alert('Local storage is unsupported in this browser');
  return;
}
```

You can enable console warning if set fails with `sessioncache.enableWarnings()`:

```js
// enable warnings
sessioncache.enableWarnings(true);

// disable warnings
sessioncache.enableWarnings(false);
```

The library also takes care of serializing objects, so you can store more complex data:

```js
sessioncache.set('data', {'name': 'Pamela', 'age': 26}, 2);
```

And then when you retrieve it, you will get it back as an object:

```js
alert(sessioncache.get('data').name);
```

If you have multiple instances of sessioncache running on the same domain, you can partition data in a certain bucket via:

```js
sessioncache.set('response', '...', 2);
sessioncache.setBucket('lib');
sessioncache.set('path', '...', 2);
sessioncache.flush(); //only removes 'path' which was set in the lib bucket
```

For more live examples, play around with the demo here:
http://pamelafox.github.com/sessioncache/demo.html


Real-World Usage
----------
This library was originally developed with the use case of caching results of JSON API queries
to speed up my webapps and give them better protection against flaky APIs.
(More on that in this [blog post](http://blog.pamelafox.org/2010/10/sessioncache-sessionStorage-based-memcache.html))

For example, [RageTube](http://ragetube.net) uses `sessioncache` to fetch Youtube API results for 10 minutes:

```js
var key = 'youtube:' + query;
var json = sessioncache.get(key);
if (json) {
  processJSON(json);
} else {
  fetchJSON(query);
}

function processJSON(json) {
  // ..
}

function fetchJSON() {
  var searchUrl = 'http://gdata.youtube.com/feeds/api/videos';
  var params = {
   'v': '2', 'alt': 'jsonc', 'q': encodeURIComponent(query)
  }
  JSONP.get(searchUrl, params, null, function(json) {
    processJSON(json);
    sessioncache.set(key, json, 10);
  });
}
```

It does not have to be used for only expiration-based caching, however. It can also be used as just a wrapper for `sessionStorage`, as it provides the benefit of handling JS object (de-)serialization.

For example, the [QuizCards](http://quizcards.info) Chrome extensions use `sessioncache`
to store the user statistics for each user bucket, and those stats are an array
of objects.

```js
function initBuckets() {
  var bucket1 = [];
  for (var i = 0; i < CARDS_DATA.length; i++) {
    var datum = CARDS_DATA[i];
    bucket1.push({'id': datum.id, 'lastAsked': 0});
  }
  sessioncache.set(LS_BUCKET + 1, bucket1);
  sessioncache.set(LS_BUCKET + 2, []);
  sessioncache.set(LS_BUCKET + 3, []);
  sessioncache.set(LS_BUCKET + 4, []);
  sessioncache.set(LS_BUCKET + 5, []);
  sessioncache.set(LS_INIT, 'true')
}
```

Browser Support
----------------

The `sessioncache` library should work in all browsers where `sessionStorage` is supported.
A list of those is here:
http://www.quirksmode.org/dom/html5.html

