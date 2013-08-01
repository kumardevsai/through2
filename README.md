# through2

[![Build Status](https://secure.travis-ci.org/rvagg/through2.png)](http://travis-ci.org/rvagg/through2)

[![NPM](https://nodei.co/npm/through2.png?compact=true)](https://nodei.co/npm/through2/) 

**A tiny wrapper around Node streams.Transform (Streams2) to avoid explicit subclassing noise**

Inspired by [Dominic Tarr](https://github.com/dominictarr)'s [through](https://github.com/dominictarr/through) in that it's so much easier to make a stream out of a function than it is to set up the prototype chain properly: `through(function (chunk) { ... })`.

```js
fs.createReadStream('ex.txt')
    .pipe(through2(function (chunk, enc, callback) {
    for (var i = 0; i < chunk.length; i++)
      if (chunk[i] == 97)
        chunk[i] = 122 // swap 'a' for 'z'
    this.push(chunk)
    callback()
  }))
  .pipe(fs.createWriteStream('out.txt'))
```

Or object streams:

```js
var all = []
fs.createReadStream('data.csv')
  .pipe(csv2())
  .pipe(through2({ objectMode: true }, function (chunk, enc, callback) {
  	var data = {
        name    : chunk[0]
      , address : chunk[3]
      , phone   : chunk[10]
    }
    this.push(data)
    callback()
  }))
  .on('data', function (data) {
    all.push(data)
  })
  .on('end', function () {
    doSomethingSpecial(all)
  })
```

## API

<b><code>through2([ options, ] transformFunction[, flushFunction ])</code></b>

Consult the **[stream.Transform](http://nodejs.org/docs/latest/api/stream.html#stream_class_stream_transform)** documentation for the exact rules of the `transformFunction` (i.e. `this._transform`) and the optional `flushFunction` (i.e. `this._flush`).

### options

The options argument is optional and is passed straight through to `stream.Transform`. So you can use `objectMode:true` if you are processing non-binary streams.

The `options` argument is first, unlike standard convention, because if I'm passing in an anonymous function then I'd prefer for the options argument to not get lost at the end of the call:

```js
fs.createReadStream('/tmp/important.dat')
	.pipe(through2({ objectMode: true, allowHalfOpen: false }, function (chunk, enc, cb) {
		this.push(new Buffer('wut?'))
  	cb()
	})
  .pipe(fs.createWriteStream('/tmp/wut.txt'))
```

### transformFunction

The `transformFunction` must have the following signature: `function (chunk, encoding, callback) {}`. A minimal implementation should call the `callback` function to indicate that the transformation is done, even if that transformation means discarding the chunk.

To queue a new chunk, call `this.push(chunk)`&mdash;this can be called as many times as required before the `callback()` if you have multiple pieces to send on.

### flushFunction

The optional `flushFunction` is provided as the last argument (2nd or 3rd, depending on whether you've supplied options) is called just prior to the stream ending. Can be used to finish up any processing that may be in progress.


## License

**through2** is Copyright (c) 2013 Rod Vagg [@rvagg](https://twitter.com/rvagg) and licenced under the MIT licence. All rights not explicitly granted in the MIT license are reserved. See the included LICENSE file for more details.