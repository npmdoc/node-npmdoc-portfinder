# api documentation for  [portfinder (v1.0.13)](https://github.com/indexzero/node-portfinder#readme)  [![npm package](https://img.shields.io/npm/v/npmdoc-portfinder.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-portfinder) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-portfinder.svg)](https://travis-ci.org/npmdoc/node-npmdoc-portfinder)
#### A simple tool to find an open port on the current machine

[![NPM](https://nodei.co/npm/portfinder.png?downloads=true)](https://www.npmjs.com/package/portfinder)

[![apidoc](https://npmdoc.github.io/node-npmdoc-portfinder/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-portfinder_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-portfinder/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-portfinder/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-portfinder/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Charlie Robbins",
        "email": "charlie.robbins@gmail.com"
    },
    "bugs": {
        "url": "https://github.com/indexzero/node-portfinder/issues"
    },
    "dependencies": {
        "async": "^1.5.2",
        "debug": "^2.2.0",
        "mkdirp": "0.5.x"
    },
    "description": "A simple tool to find an open port on the current machine",
    "devDependencies": {
        "glob": "^6.0.4",
        "vows": "0.8.0"
    },
    "directories": {},
    "dist": {
        "shasum": "bb32ecd87c27104ae6ee44b5a3ccbf0ebb1aede9",
        "tarball": "https://registry.npmjs.org/portfinder/-/portfinder-1.0.13.tgz"
    },
    "engines": {
        "node": ">= 0.12.0"
    },
    "files": [
        "lib"
    ],
    "gitHead": "20161e8e00355099c90062a069a7aa68dc9bcf9b",
    "homepage": "https://github.com/indexzero/node-portfinder#readme",
    "keywords": [
        "http",
        "ports",
        "utilities"
    ],
    "license": "MIT",
    "main": "./lib/portfinder",
    "maintainers": [
        {
            "name": "eriktrom",
            "email": "erik.trom.github@gmail.com"
        },
        {
            "name": "indexzero",
            "email": "charlie.robbins@gmail.com"
        }
    ],
    "name": "portfinder",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+ssh://git@github.com/indexzero/node-portfinder.git"
    },
    "scripts": {
        "test": "vows test/*-test.js --spec"
    },
    "types": "./lib/portfinder.d.ts",
    "version": "1.0.13"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module portfinder](#apidoc.module.portfinder)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>getPort (options, callback)](#apidoc.element.portfinder.getPort)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>getPortPromise (options)](#apidoc.element.portfinder.getPortPromise)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>getPorts (count, options, callback)](#apidoc.element.portfinder.getPorts)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>getSocket (options, callback)](#apidoc.element.portfinder.getSocket)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>nextPort (port)](#apidoc.element.portfinder.nextPort)
1.  [function <span class="apidocSignatureSpan">portfinder.</span>nextSocket (socketPath)](#apidoc.element.portfinder.nextSocket)
1.  number <span class="apidocSignatureSpan">portfinder.</span>basePort
1.  object <span class="apidocSignatureSpan">portfinder.</span>_defaultHosts
1.  string <span class="apidocSignatureSpan">portfinder.</span>basePath



# <a name="apidoc.module.portfinder"></a>[module portfinder](#apidoc.module.portfinder)

#### <a name="apidoc.element.portfinder.getPort"></a>[function <span class="apidocSignatureSpan">portfinder.</span>getPort (options, callback)](#apidoc.element.portfinder.getPort)
- description and source-code
```javascript
getPort = function (options, callback) {
  if (!callback) {
    callback = options;
    options = {};
  }

  if (options.host) {

    var hasUserGivenHost;
    for (var i = 0; i < exports._defaultHosts.length; i++) {
      if (exports._defaultHosts[i] === options.host) {
        hasUserGivenHost = true;
        break;
      }
    }

    if (!hasUserGivenHost) {
      exports._defaultHosts.push(options.host);
    }

  }

  var openPorts = [], currentHost;
  return async.eachSeries(exports._defaultHosts, function(host, next) {
    debugGetPort("in eachSeries() iteration callback: host is", host);

    return internals.testPort({ host: host, port: options.port }, function(err, port) {
      if (err) {
        debugGetPort("in eachSeries() iteration callback testPort() callback", "with an err:", err.code);
        currentHost = host;
        return next(err);
      } else {
        debugGetPort("in eachSeries() iteration callback testPort() callback",
                    "with a success for port", port);
        openPorts.push(port);
        return next();
      }
    });
  }, function(err) {

    if (err) {
      debugGetPort("in eachSeries() result callback: err is", err);
      // If we get EADDRNOTAVAIL it means the host is not bindable, so remove it
      // from exports._defaultHosts and start over. For ubuntu, we use EINVAL for the same
      if (err.code === 'EADDRNOTAVAIL' || err.code === 'EINVAL') {
        if (options.host === currentHost) {
          // if bad address matches host given by user, tell them
          //
          // NOTE: We may need to one day handle 'my-non-existent-host.local' if users
          // report frustration with passing in hostnames that DONT map to bindable
          // hosts, without showing them a good error.
          var msg = 'Provided host ' + options.host + ' could NOT be bound. Please provide a different host address or hostname';
          return callback(Error(msg));
        }

        var idx = exports._defaultHosts.indexOf(currentHost);
        exports._defaultHosts.splice(idx, 1);
        return exports.getPort(options, callback);
      } else {
        // error is not accounted for, file ticket, handle special case
        return callback(err);
      }
    }

    // sort so we can compare first host to last host
    openPorts.sort(function(a, b) {
      return a - b;
    });

    debugGetPort("in eachSeries() result callback: openPorts is", openPorts);

    if (openPorts[0] === openPorts[openPorts.length-1]) {
      // if first === last, we found an open port
      return callback(null, openPorts[0]);
    } else {
      // otherwise, try again, using sorted port, aka, highest open for >= 1 host
      return exports.getPort({ port: openPorts.pop(), host: options.host }, callback);
    }

  });
}
```
- example usage
```shell
...

## Usage
The 'portfinder' module has a simple interface:

''' js
  var portfinder = require('portfinder');

  portfinder.getPort(function (err, port) {
    //
    // 'port' is guaranteed to be a free port
    // in this scope.
    //
  });
'''
...
```

#### <a name="apidoc.element.portfinder.getPortPromise"></a>[function <span class="apidocSignatureSpan">portfinder.</span>getPortPromise (options)](#apidoc.element.portfinder.getPortPromise)
- description and source-code
```javascript
getPortPromise = function (options) {
  if (typeof Promise !== 'function') {
    throw Error('Native promise support is not available in this version of node.' +
      'Please install a polyfill and assign Promise to global.Promise before calling this method');
  }
  if (!options) {
    options = {};
  }
  return new Promise(function(resolve, reject) {
    exports.getPort(options, function(err, port) {
      if (err) {
        return reject(err);
      }
      resolve(port);
    });
  });
}
```
- example usage
```shell
...
'''

Or with promise (if Promise are supported) :

''' js
const portfinder = require('portfinder');

portfinder.getPortPromise()
  .then((port) => {
      //
      // 'port' is guaranteed to be a free port
      // in this scope.
      //
  })
  .catch((err) => {
...
```

#### <a name="apidoc.element.portfinder.getPorts"></a>[function <span class="apidocSignatureSpan">portfinder.</span>getPorts (count, options, callback)](#apidoc.element.portfinder.getPorts)
- description and source-code
```javascript
getPorts = function (count, options, callback) {
  if (!callback) {
    callback = options;
    options = {};
  }

  var lastPort = null;
  async.timesSeries(count, function(index, asyncCallback) {
    if (lastPort) {
      options.port = exports.nextPort(lastPort);
    }

    exports.getPort(options, function (err, port) {
      if (err) {
        asyncCallback(err);
      } else {
        lastPort = port;
        asyncCallback(null, port);
      }
    });
  }, callback);
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.portfinder.getSocket"></a>[function <span class="apidocSignatureSpan">portfinder.</span>getSocket (options, callback)](#apidoc.element.portfinder.getSocket)
- description and source-code
```javascript
getSocket = function (options, callback) {
  if (!callback) {
    callback = options;
    options = {};
  }

  options.mod  = options.mod    || parseInt(755, 8);
  options.path = options.path   || exports.basePath + '.sock';

  //
  // Tests the specified socket
  //
  function testSocket () {
    fs.stat(options.path, function (err) {
      //
      // If file we're checking doesn't exist (thus, stating it emits ENOENT),
      // we should be OK with listening on this socket.
      //
      if (err) {
        if (err.code == 'ENOENT') {
          callback(null, options.path);
        }
        else {
          callback(err);
        }
      }
      else {
        //
        // This file exists, so it isn't possible to listen on it. Lets try
        // next socket.
        //
        options.path = exports.nextSocket(options.path);
        exports.getSocket(options, callback);
      }
    });
  }

  //
  // Create the target 'dir' then test connection
  // against the socket.
  //
  function createAndTestSocket (dir) {
    mkdirp(dir, options.mod, function (err) {
      if (err) {
        return callback(err);
      }

      options.exists = true;
      testSocket();
    });
  }

  //
  // Check if the parent directory of the target
  // socket path exists. If it does, test connection
  // against the socket. Otherwise, create the directory
  // then test connection.
  //
  function checkAndTestSocket () {
    var dir = path.dirname(options.path);

    fs.stat(dir, function (err, stats) {
      if (err || !stats.isDirectory()) {
        return createAndTestSocket(dir);
      }

      options.exists = true;
      testSocket();
    });
  }

  //
  // If it has been explicitly stated that the
  // target 'options.path' already exists, then
  // simply test the socket.
  //
  return options.exists
    ? testSocket()
    : checkAndTestSocket();
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.portfinder.nextPort"></a>[function <span class="apidocSignatureSpan">portfinder.</span>nextPort (port)](#apidoc.element.portfinder.nextPort)
- description and source-code
```javascript
nextPort = function (port) {
  return port + 1;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.portfinder.nextSocket"></a>[function <span class="apidocSignatureSpan">portfinder.</span>nextSocket (socketPath)](#apidoc.element.portfinder.nextSocket)
- description and source-code
```javascript
nextSocket = function (socketPath) {
  var dir = path.dirname(socketPath),
      name = path.basename(socketPath, '.sock'),
      match = name.match(/^([a-zA-z]+)(\d*)$/i),
      index = parseInt(match[2]),
      base = match[1];

  if (isNaN(index)) {
    index = 0;
  }

  index += 1;
  return path.join(dir, base + index + '.sock');
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
