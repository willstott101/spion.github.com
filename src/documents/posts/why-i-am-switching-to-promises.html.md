---
title: Why I am switching to promises
layout: post
hidden: true
date: 2013-10-07
---

I'm switching my node code from callbacks to promises. The reasons aren't
merely aesthetical, they're rather practical:


<a name="exception-crash"></a>
### Exception? Crash!

We're all human. We make mistakes, and then JavaScript `throw`s an error. How 
do callbacks punish mistakes? They crash your process! 

> But spion, why don't you use domains?

Yes, I could do that. I could [crash my process gracefully][must-shutdown]
instead of letting it just crash. But its still a crash no matter what
lipstick you put on it. With thousands of users, 0.5% hitting a throwing
path means over 50 process shutdowns and most likely denial of service.

Promises are exception safe. If an unhandled exception happens in one of the 
`.then` callbacks, only that promise chain will die. I can also attach an error 
handler to handle the exception and do any clean up if necessary - 
transparently! The process will happily continue to serve the rest of my users.

### `if (err) return done(err)`

That line is haunting me in my dreams now. I'm honestly tired of writing it.
What happened to the [DRY principle](dry)?

> But spion, why don't you wrap all your calbacks? 

I guess I could do that and lose the callback stack when generating a 
`new Error()`. Or since I'm already wrapping things, why not wrap the
entire thing with promises, rely on longStackSupport, and handle errors at
at my discretion?

### [Releasing Zalgo][releasing-zalgo]

Your promise library prevents you from [releasing Zalgo][releasing-zalgo]. You 
can't release Zalgo with promises. Its impossible for a promise to result with 
the release of the Zalgo-beast. [Promises are Zalgo-safe (see section 
3.1)][promises-zalgo-safe].

### Promises are now part of ES6

Yes, they will become a part of the language. New DOM APIs will be using them 
too. jQuery already uses promises..ish things, angular utilizes promises 
everywhere (even in the templates), and Ember uses promises extensively

Browser code has already switched. I'm switching too.

## But promises are slow!

Yeah, I know I wrote that. But I was wrong. A month after I wrote 
[the giant comparison of async patterns][the-analysis], Petka Antonov wrote 
[Bluebird][bluebird]. Its a wicked fast promise library, and here are the 
charts to prove it:

Run time (ms)

<div id="perf-time-promises" class="plot">
</div>
<script type="text/javascript">


window.perfTimePromises = 
[ { label: 'callbacks-async-waterfall.js',
    data: 
     [ [ '200', 19 ],
       [ '500', 31 ],
       [ '1000', 88 ],
       [ '2000', 146 ],
       [ '5000', 400 ],
       [ '10000', 881 ],
       [ '20000', 1826 ] ] },
  { label: 'promises-bluebird-generator.js',
    data: 
     [ [ '200', 13 ],
       [ '500', 16 ],
       [ '1000', 34 ],
       [ '2000', 69 ],
       [ '5000', 202 ],
       [ '10000', 392 ],
       [ '20000', 891 ] ] },
  { label: 'promises-bluebird.js',
    data: 
     [ [ '200', 12 ],
       [ '500', 20 ],
       [ '1000', 43 ],
       [ '2000', 96 ],
       [ '5000', 278 ],
       [ '10000', 538 ],
       [ '20000', 1021 ] ] },
  { label: 'callbacks-flattened.js',
    data: 
     [ [ '200', 20 ],
       [ '500', 39 ],
       [ '1000', 53 ],
       [ '2000', 99 ],
       [ '5000', 166 ],
       [ '10000', 342 ],
       [ '20000', 654 ] ] },
  { label: 'callbacks-generator-suspend.js',
    data: 
     [ [ '200', 16 ],
       [ '500', 18 ],
       [ '1000', 36 ],
       [ '2000', 69 ],
       [ '5000', 227 ],
       [ '10000', 490 ],
       [ '20000', 968 ] ] },
  { label: 'thunks-generator-gens.js',
    data: 
     [ [ '200', 35 ],
       [ '500', 46 ],
       [ '1000', 71 ],
       [ '2000', 111 ],
       [ '5000', 272 ],
       [ '10000', 569 ],
       [ '20000', 1000 ] ] } ]

window.addEventListener('load', function() {
    $.plot('#perf-time-promises', window.perfTimePromises, {legend: { position: 'nw' }});
});

</script>

Also, memory usage (MB)

<div id="perf-mem-promises" class="plot">
</div>
<script type="text/javascript">

window.perfMemPromises = 

[ { label: 'callbacks-async-waterfall.js',
    data: 
     [ [ '200', 1.76953125 ],
       [ '500', 5.5390625 ],
       [ '1000', 9.8515625 ],
       [ '2000', 20.81640625 ],
       [ '5000', 55.79296875 ],
       [ '10000', 70.703125 ],
       [ '20000', 153.83203125 ] ] },
  { label: 'promises-bluebird-generator.js',
    data: 
     [ [ '200', 0.80859375 ],
       [ '500', 1.02734375 ],
       [ '1000', 3.08203125 ],
       [ '2000', 6.8046875 ],
       [ '5000', 18.91015625 ],
       [ '10000', 41.828125 ],
       [ '20000', 60.2734375 ] ] },
  { label: 'promises-bluebird.js',
    data: 
     [ [ '200', 0.78515625 ],
       [ '500', 1.1171875 ],
       [ '1000', 5 ],
       [ '2000', 10.41015625 ],
       [ '5000', 33.3046875 ],
       [ '10000', 57.0546875 ],
       [ '20000', 105.953125 ] ] },
  { label: 'callbacks-flattened.js',
    data: 
     [ [ '200', 0.28515625 ],
       [ '500', 1.5703125 ],
       [ '1000', 2.91015625 ],
       [ '2000', 6.15234375 ],
       [ '5000', 18.015625 ],
       [ '10000', 19.671875 ],
       [ '20000', 59.41015625 ] ] },
  { label: 'callbacks-generator-suspend.js',
    data: 
     [ [ '200', 0.984375 ],
       [ '500', 1.03125 ],
       [ '1000', 3.8671875 ],
       [ '2000', 7.33203125 ],
       [ '5000', 21.640625 ],
       [ '10000', 44.99609375 ],
       [ '20000', 65.28125 ] ] },
  { label: 'thunks-generator-gens.js',
    data: 
     [ [ '200', 1.03125 ],
       [ '500', 2.14453125 ],
       [ '1000', 4.86328125 ],
       [ '2000', 9.8125 ],
       [ '5000', 23.09375 ],
       [ '10000', 41.83984375 ],
       [ '20000', 57.10546875 ] ] } ]


window.addEventListener('load', function() {
    $.plot('#perf-mem-promises', perfMemPromises, {legend: { position: 'nw' }, 
		yaxis: {min: 0}});
});

</script>

Here is a bigger table for 10 000 parallel operations, 1 ms per I/O op.
Measure ALL the things!

| file                                       | time(ms) | memory(MB) |
|:-------------------------------------------|---------:|-----------:|
| callbacks-original.js                      |      316 |      34.97 |
| callbacks-flattened.js                     |      335 |      35.10 |
| callbacks-catcher.js                       |      355 |      30.20 |
| promises-bluebird-generator.js             |      364 |      41.89 |
| dst-streamline.js                          |      441 |      46.91 |
| callbacks-deferred-queue.js                |      455 |      38.10 |
| callbacks-generator-suspend.js             |      466 |      45.20 |
| promises-bluebird.js                       |      512 |      57.45 |
| thunks-generator-gens.js                   |      517 |      40.29 |
| thunks-generator-co.js                     |      707 |      47.95 |
| promises-compose-bluebird.js               |      710 |      73.11 |
| callbacks-generator-genny.js               |      801 |      67.67 |
| callbacks-async-waterfall.js               |      989 |      89.97 |
| promises-bluebird-spawn.js                 |     1227 |      66.98 |
| promises-kew.js                            |     1578 |     105.14 |
| dst-stratifiedjs-compiled.js               |     2341 |     148.24 |
| rx.js                                      |     2369 |     266.59 |
| promises-when.js                           |     7950 |     240.11 |
| promises-q-generator.js                    |    21828 |     702.93 |
| promises-q.js                              |    28262 |     712.93 |
| promises-compose-q.js                      |    59413 |     778.05 |

Promises are not slow. At least, not anymore. Infact, bluebird generators
are almost as fast as regular callback code (and the fastest generators so
far). And bluebird promises are definitely faster than using `async.waterfall`

Considering that bluebird wraps the underlying callback-using libraries **and**
makes your own callbacks exception-safe, this is really amazing.

note: `async.waterfall` doesn't do this. exceptions still crash your process.

## What about stack traces?

Bluebird has them behind a flag that has slows it down about 5 times. They're
even longer than Q's `longStackSupport`: bluebird can give you the entire event
chain. Simply enable the flag in development mode, and you're suddenly in 
debugging nirvana.

## What about the community? 

This is a valid point. Mikeal said it: If you write a library based on promises, 
[nobody is going to use it][mikeal-talk].

However, both bluebird and Q give you `promise.nodeify`. With it, you can write 
a library with a dual API that can both take callbacks and return promises:

```js
module.exports = function fetch(itemId, callback) {
	return locate(itemId).then(function(location) {
		return getFrom(location, itemId);
	}).nodeify(callback);
}
```

And now my library is not imposing promises on you. Infact, my library is even 
friendlier to the community: if I make a dumb mistake within the library that 
causes an exception to be thrown, it wont crash your process but instead it 
will pass that exception as an error to your callback. 

As a bonus, I don't have to fear the wrath of my angry users for crashing their 
production servers. Thats always a plus.

## What about generators?

To use generators with callbacks you have two options

1. use a resumer library like [suspend] or [genny]
2. wrap callback-taking functions to become thunk returning functions.

Since #1 is proving to be unpopular, why not just `s/thunk/promise/g` and use
generators with promises?

## But promises are unnecessarily complicated!

Promises are actually pretty simple, its just that the usual terminology used
to explain them is complicated. Here is a straight-forward guide that uses known 
principles and analogies from node (remember, the focus is on simplicity,
not correctness):

Promises are objects that have a `.then` method. The `.then` method takes 2
callbacks, a success callback and an error callback. When one of these two 
callbacks returns a value or throws an exception, `.then` must behave in a way 
that enables stream-like chaining and simplified error handling. Lets explain 
that behavior through examples:

Imagine that node's `fs` was wrapped to work in this manner. This is pretty 
easy to do - bluebird already does something like that with 
[`promisify()`][promisify]. Then this code:

```js
fs.readFile(file, function(err, res) {
    if (err) handleError();
    doStuffWith(res);
});
```

will look like this:

```js
fs.readFile(file).then(function(res) {
    doStuffWith(res);
}, function(err) {
    handleError();
});
```

Pretty much the same so far, except you use a second callback for the error 
(which isn't really better). So when does it get better?

Its better because you can attach a callback later if you want. Remember, 
`fs.readFile(file)` returns a promise now, so you can put that in a var, or
return them from your functions:

```js
var filePromise = fs.readFile(file);
// do more stuff...
filePromise.then(function(res) { ... });
```

Okay, that's still not much of an improvement. How about this then? You can 
attach more than one callback to a promise if you like:

```js
filePromise.then(function(res) { uploadData(url, res); });
filePromise.then(function(res) { saveLocal(url, res); });
```

Hey, this is beginning to look a lot like streams - they too can be piped to
multiple destinations.

Still not good enough? 

What if I told you... that if you return something from inside the .then() 
callback, then you'll get a promise for that thing?

Say you want to get a line from a file.

```js
var linePromise = fs.readFile(file).then(function(data) {
    return data.toString.split('\n')[line];
});

var beginsWithHelloPromise = linePromise.then(function(line) {
    return /^hello/.test(line);
});

var message = beginsWithHelloPromise.then(function(beginsWithHello) {
	if (beginsWithHello) return "Looks like hello world";
	else return "Probably not hello world";
});
```

Thats pretty cool, although not very useful - we could just put all sync
operations in the first `.then()` callback. 

But guess what happens when we return a promise from within that `.then` 
callback. We get a promise for that promise outside of `.then()`?  Nope, we 
just get the same promise on the outside too!

```js
function readUploadAndSave(file, url, otherPath) {
	// read the file
	var filePromise = fs.readFile(file);
	// upload it when done reading
	var uploadedPromise = filePromise.then(function(content) { 
		return uploadData(url, content);
	});
	// also save to another place locally
	var savedLocalPromise = filePromise.then(function(res) {
		return fs.saveFile(res, otherPath)
	});
	// return a promise that "succeeds" when both saving and uploading succeed:
	return Promise.join(uploadedPromise, savedLocalPromise);
}
readUploadAndSave(file, url, otherPath).then(function() {
	console.log("Success!");
}, function(err) {
	// This function will catch *ALL* errors from the above 
	// operations including any exceptions thrown inside .then 
	console.log("Oops, it failed.", err);
});
```


Now its easier to understand chaining: at the end of every function passed
to a `.then()` call, simply return a promise. Lets make our code even shorter:

```js
function readUploadAndSave(file, url, otherPath) {
	return fs.readFile(file).then(function(content) {
		// upload and save when done reading
		return Bluebird.join(uploadData(url, content), 
			fs.saveFile(res, otherPath));
});
  }

Mind = blown! Notice how I'm not handling errors if I can't handle them there.
They will automatically get passed with the returned promise.

What if we want to make sure the data is uploaded first before saving?

```js
function readUploadAndSave(file, url, otherPath) {
	var content;
	// read the file
	var uploadedPromise = fs.readFile(file).then(function(vContent)
		content = vContent;
		// return promise that its uploaded
  		return uploadData(url, content);
	});
	return uploadedPromise.then(function() {
		// return promise that its saved.
		return fs.saveFile(content, otherPath);
	});
}
```

Or even shorter, skip the temporary promise variables and chain `promise.then` 
the way we chain `stream.pipe` in node:

```js
function readUploadAndSave(file, url, otherPath) {
    var content;
    // read the file
    return fs.readFile(file).then(function(vContent)
        content = vContent;
        // then upload it
        return uploadData(url, content);
    }).then(function() { // after its uploaded
        // save it
        return fs.saveFile(content, otherPath);
    });
}
```

And similarly to how in a `stream.pipe` chain the last stream is returned, in 
promise pipes the promise returned from the last `.then` callback is returned.

Thats all you need, really. The rest is just converting callback-taking 
functions to promise-returning functions and using the stuff above to do your 
control flow.

You can also return values in case of an error. So for example, to write a
`readFileOrDefault` (which returns a default value if for example the file 
doesn't exist) you would simply return the default value from the error
callback:

```js
function readFileOrDefault(file, line, defaultContent) {
	return fs.readFile(file).then(function(fileContent) {
    	return fileContent;
    }, function(err) {
    	return defaultContent;
    });
}
```

You can also throw exceptions within both callbacks passed to `.then`. The 
user of the returned promise can catch those errors by adding the second
.then handler

Finally, you can make sure your resources are released in all cases, even
when an error or an exception happens:

```js
var result = doSomethingAsync();

return result.then(function(value) {
    // clean up first, then return the value.
	return cleanUp().then(function() { return value; })	
}, function(err) {
    // clean up, then re-throw that error
	return cleanUp().then(function() { throw err; });
})
```

Or you can do the same using `.finally` (from both Bluebird and Q):

```js
var result = doSomethingAsync();
return result.finally(cleanUp);
```

The same promise is still returned, but only after `cleanUp` completes.

## But what about [async][async]?

Since promises are actual values, most of the tools in async.js become 
unnecessary and you can just use whatever you're using for regular values, like 
your regular `array.map` / `array.reduce` functions combined with some promise 
tools like `.all` 

You already have async.waterfall and async.auto with .then and .spread 
chaining:

```js
files.getLastTwoVersions(filename)
	.then(function(items) {
		// fetch versions in parallel
		var v1 = versions.get(items.last),
		    v2 = versions.get(items.previous);
        return [v1, v2];
	})
	.spread(function(v1, v2) { 
	    // both of these are now complete.
	    return diffService.compare(v1.blob, v2.blob)
	})
	.then(function(diff) {
	    // voila, diff is ready. Do something with it.
	});
```

`async.parallel` / `async.map` are straightforward:

```js
// download all items, then get their names
var pNames = ids.map(function(id) { 
    return getItem(id).then(function(result) { 
    	return result.name;
    });
});
// wait for things to complete:
Promise.all(pNames).then(function(names) {
    // we now have all the names.
});
```

What if you want to wait for the current item to download first (like 
`async.mapSeries` and `async.series`)? Thats also pretty straightforward: just 
wait for the current download to complete, then start the next download, then 
extract the item name, and thats exactly what you say in the code:

```js
// start with current being an "empty" already-fulfilled promise
var current = Promise.fulfilled();
var namePromises = ids.map(function(id) { 
    // wait for the current download to complete, then get the next
    // item, then extract its name.
    current = current
        .then(function() { return getItem(id); })
        .then(function(item) { return item.name; });
    return current;
}); 
Promise.all(namePromises).then(function(names) {
    // use all names here.
});
```

The only thing that remains is mapLimit - which is a bit harder to write - but 
still not that hard:

```js
var queued = [], parallel = 3;
var namePromises = ids.map(function(id) {
    // How many items must be complete before fetching the next?
    // The queued, minus those running in parallel, plus one of 
    // the parallel slots.
    var minComplete = Math.max(0, queued.length - parallel + 1);
    // when enough items are complete, queue another request for an item    
    return Promise.some(queued, minComplete)
        .then(function() {
            var download = getItem(id);
            queued.push(download);
        	return download;        
        }).then(function(item) {
            // after that new download completes, get the item's name.	
        	return item.name;
        });
  });
Promise.all(namePromises).then(function(names) {
    // use all names here.
});
```

That covers most of async. But we've barely started to scratch the surface.

For example, promises could work very well with streams. Imagine a `limit` 
stream that allows at most 3 promises resolving in parallel, backpressuring 
otherwise, processing items from leveldb:

```js
originalSublevel.createReadStream().pipe(limit(3, function(data) {
	return convertor(data.value).then(function(converted) {
		return {key: data.key, value: converted};
	});
})).pipe(convertedSublevel.createWriteStream());
```

Looks awesome. I definitely want to explore that.

[bluebird]: https://github.com/petkaantonov/bluebird
[releasing-zalgo]: http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony
[the-analysis]: /posts/analysis-generators-and-other-async-patterns-node.html
[must-shutdown]: http://nodejs.org/api/domain.html#domain_warning_don_t_ignore_errors
[dry]: http://c2.com/cgi/wiki?DontRepeatYourself
[promises-zalgo-safe]: http://promisesaplus.com/#point-87
[mikeal-talk]: http://www.youtube.com/watch?v=GaqxIMLLOu8
[promisify]: https://github.com/petkaantonov/bluebird/blob/master/API.md#promisepromisifyobject-target---object
[async]:https://github.com/caolan/async
[suspend]:https://github.com/jmar777/suspend
[genny]:https://github.com/jmar777/genny

<script src="/scripts/jquery.flot.js"></script>
<script src="/scripts/jquery.flot.highlightSeries.js"></script>
