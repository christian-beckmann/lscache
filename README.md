sessioncache
===============================
This is a simple library that emulates `memcache` functions using HTML5 `sessionStorage`, so that you can cache data on the client
and associate an expiration time with each piece of data. If the `sessionStorage` limit (~10MB) is exceeded, it tries to create space by removing the items that are closest to expiring anyway. If `sessionStorage` is not available at all in the browser, the library degrades by simply not caching and all cache requests return null.

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


Browser Support
----------------

The `sessioncache` library should work in all browsers where `sessionStorage` is supported.
A list of those is here:
http://caniuse.com/#feat=namevalue-storage

Further docs:
https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API