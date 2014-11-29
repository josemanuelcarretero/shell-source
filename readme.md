# shell-source
Source environment variables from a shell script into a [Node.js](http://nodejs.org/) process.

#### Dragons:
> Since sourcing a shell script means actually executing code, it is an unsafe thing to do unless you can 100% guarantee its contents are not `rm -rf /`. You've been warned!

## Why
You have some configuration data stored in a sourcable shell script, but need access to that data from a JavaScript program. You could try to parse the file as text, but that would only work if you could be sure the script does not expand any variables or execute any code.

## How
Spawns the process owner's default shell and executes a POSIX compliant wrapper script that in turn [sources](http://www.tldp.org/HOWTO/Bash-Prompt-HOWTO/x237.html) the file of your choosing. The wrapper then calls `env` which writes the child process' updated environment to stdout. The parent (Node.js) process can then parse this output and update `process.env` accordingly.

## Example
Consider the (contrived) example below of a script that needs to be executed before its variables can be evaluated in a useful way:
```bash
export SERVER_HOST="$(hostname)"
export SERVER_PORT="$(grep -m1 '# HTTP Alternate' < /etc/services | sed 's/[^0-9]*\(.*\)\/.*/\1/')"
export PATH="node_modules/.bin:$PATH"
```

A Node.js process can then use `shell-source` to emulate the behavior of `sh`'s `.` built-in, executing the script before absorbing any enviroment changes it effects:
```javascript
var source = require('shell-source');

source(__dirname + '/env.sh', function(err) {
  if (err) return console.error(err);

  console.log(process.env.SERVER_HOST); // ::
  console.log(process.env.SERVER_PORT); // 8080
  console.log(process.env.PATH);        // node_modules/.bin:/usr/local/bin
});
```

## Install
```bash
$ npm install shell-source
```

## Test
```bash
$ npm test
```

## Require
#### `var source = require('shell-source');`

## Use
#### `source(filepath, [opts,] callback);`
* `filepath` The full path to the shell script that should be sourced.
* `opts` An options object which can contain:
	* `source` A boolean. Defaults to `true`. If set to `false`, the callback can receive the evironment object as its second argument and `process.env` will be left unmolested.
* `callback` A callback with signature:
```javascript
function(err, environment) {
	console.log(err, environment);
}
```

## Notes
Obviously it would be really nice if this could be done synchronously. However, until something like [this](http://strongloop.com/strongblog/whats-new-in-node-js-v0-12-execsync-a-synchronous-api-for-child-processes) lands on stable, I'm not sure if there is a sane way to accomplish it. If there is, holla.

## Releases
The latest stable version is published to [npm](https://www.npmjs.org/package/shell-source).
* [1.0.0](https://github.com/jessetane/shell-source/releases/tag/1.0.0)
 * First release.

## License
Copyright © 2014 Jesse Tane <jesse.tane@gmail.com>

This work is free. You can redistribute it and/or modify it under the
terms of the [WTFPL](http://www.wtfpl.net/txt/copying).

No Warranty. The Software is provided "as is" without warranty of any kind, either express or implied, including without limitation any implied warranties of condition, uninterrupted use, merchantability, fitness for a particular purpose, or non-infringement.