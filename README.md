crx
===

[![Build Status](https://secure.travis-ci.org/jed/crx.png)](http://travis-ci.org/jed/crx)

crx is a [node.js](http://nodejs.org/) command line app for packing Google Chrome extensions. If you'd like to integrate it into your [grunt](http://gruntjs.com/) workflow, give [oncletom](https://github.com/oncletom)'s [grunt-crx](https://github.com/oncletom/grunt-crx) a spin.

## Requirements

* [node.js](http://nodejs.org/), tested with 0.4.12
* openssl
* ssh-keygen
* zip

## Install

    $ npm install crx

## Module API

### ChromeExtension = require("crx")
### crx = new ChromeExtension(attrs)

This module exports the `ChromeExtension` constructor directly, which can take an optional attribute object, which is used to extend the instance.

### crx.load(path, callback)

Loads the Chrome Extension from the specified path.

### crx.pack(callback)

Packs the Chrome Extension, and calls back with a Buffer containing the `.crx` file.

### crx.generateUpdateXML()

Returns a Buffer containing the update.xml file used for autoupdate, as specified for `update_url` in the manifest. In this case, the instance must have a property called `codebase`.

### crx.destroy()

Destroys all of the temporary resources used for packing.

## Module example

```javascript
var fs = require("fs")
  , ChromeExtension = require("crx")
  , crx = new ChromeExtension(
      codebase: "http://localhost:8000/myFirstExtension.crx",
      privateKey: fs.readFileSync(__dirname + "/key.pem"),
      rootDirectory: __dirname + "/myFirstExtension"
    })

crx.load(function(err) {
  if (err) throw err

  this.pack(function(err, data){
    if (err) throw err

    var updateXML = this.generateUpdateXML()

    fs.writeFile(__dirname + "/update.xml", updateXML)
    fs.writeFile(__dirname + "/myFirstExtension.crx", data)
  
    this.destroy()
  })
})
```

## CLI API

### crx pack [directory] [-f file] [-p private-key]

Pack the specified directory into a .crx package, and output it to stdout. If no directory is specified, the current working directory is used.

Use the `-f` option to output to a file instead of stdout; if no file is specified, the package is given the same name as the directory basename.

Use the `-p` option to specify an external private key. If this is not used, `key.pem` is used from within the directory. If this option is not used and no `key.pem` file exists, one will be generated automatically.

Use the `-b` option to specify the maximum buffer allowed to generate extension. By default, will rely on `node` internal setting (~200KB).

### crx keygen [directory]

Generate a 1,024-bit RSA private key within the directory. This is called automatically if a key is not specified, and `key.pem` does not exist.

### crx -h

Show information about using this utility, generated by [commander](https://github.com/visionmedia/commander.js).

## CLI example

Given the following directory structure:

    └─┬ myFirstExtension
      ├── manifest.json
      └── icon.png

run this:

    cd myFirstExtension
    crx pack -f

to generate this:

    ├─┬ myFirstExtension
    │ ├── manifest.json
    │ ├── icon.png
    │ └── key.pem
    └── myFirstExtension.crx

You can also name the output file like this:

    cd myFirstExtension
    crx pack -f myFirstExtension.crx

to get the same results, or also pipe to the file manually like this.

    cd myFirstExtension
    crx pack > ../myFirstExtension.crx

As you can see a key is generated for you at `key.pem` if none exists. You can also specify an external key. So if you have this:

    ├─┬ myFirstExtension
    │ ├── manifest.json
    │ └── icon.png
    └── myPrivateKey.pem

you can run this:

    crx pack myFirstExtension -p myPrivateKey.pem -f

to sign your package without keeping the key in the directory.

Copyright
---------

Copyright (c) 2012 Jed Schmidt. See LICENSE.txt for details.

Send any questions or comments [here](http://twitter.com/jedschmidt).