# A bug in [browserify-shim](https://github.com/thlorenz/browserify-shim)?

When you use browserify-shim to make a library available globally -- let's say jQuery -- the global library isn't available inside the node_modules folder for modules that contain their own package.json.

For example, given the following structure (which is pretty close to the structure if this repo):

```
node_modules/
    foo/
       index.js
       package.json
app.js
jquery.js
package.json
```
Assume that inside the main `./package.json` we have browserify and browserify-shim set up as follows:

```
"browser": {
  "jquery": "./assets/jquery.js"
},
"browserify": {
  "transform": ["browserify-shim"]
},
"browserify-shim": {
  "jquery": "$"
}
```

You would expect `require('jquery')` to just work, thanks to the shim.  And it does, inside `app.js`.  But if `app.js` requires `foo` (which lives inside the node_modules folder), and `foo` includes a `require('jquery')`, browserify won't use the globally shimmed `jquery`.

If you remove `node_modules/foo/package.json`, browserify will complete successfully. But as soon as the `foo` module has its own `package.json`, it loses access to the shimmed `jquery`.

## To test

Clone this repository:

`git clone https://github.com/mlms13/shim-bug.git`

Install dependencies (browserify and browserify-shim)

`npm install`

With browserify installed globally, try to run the following:

`browserify index.js -o bundle.js`

You should see an error along the lines of `Error: module "jquery" not found from "/Users/<username>/Projects/shim-bug/node_modules/foo/index.js"`

If you delete `node_modules/foo/package.json` and re-run the `browserify` command, everything should work as expected.
