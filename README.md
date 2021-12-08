# esmodulestest

Trying out esmodules (npx serve)

## ESModules Notes

ESModules

- CommonJS and AMD-based module systems like RequireJS and Webpack/Babel - now browsers support module functionality natively
- Great cross browser compatibility except for IE, Firefox not yet supporting imports in workers
- Recommended to create files as .mjs for clarity and ensures module files are parsed by runtimes such as Node and build tools like Babel
- Moduels use strict mode automatically; deferred automatically so no need for “defer” script attribute; imported into scope of script they are imported into (not global scope access through console)
- dynamic module loading with import(…).then((module) => { }) to load only when needed
- import { … } from ‘…’; import defaultExport from ‘…’; import _ from ‘…’; import _ as Module from ‘…’
- top level await available in modules to act as big async functions, meaning code can be evaluated before use in parent modules but without blocking sibling modules from loading
- need to test on server rather than file:// URL due to module security concerns and need server to return back MIME type text/javascript - can do npx serve
- need to add extension for imports
- Construction - find, download and parse all of files into module records
  - Figure out where to download the file containing the module from (module resolution)
  - Fetch the file by downloading it from a URL or loading it form the file system
  - Parse the file into a module record
  - Finds entry point file; tell loader where to find it by using a script tag
  - <script src=“main.js” type=“module”>
      - imports within main to get the rest of the files
      - fetch js -> parse, form module record -> fetch more js and parse more module records for the imports
  - split algorithm into phases between construction and instantiating to avoid blocking main thread
    - Commonjs deals with filesystem rather than internet so it can block main thread while it loads the file and focuses on instantiation and evaluation (evaluates JS up to require() statements and synchronously load/evaluate file and dependencies)
    - Can use variables in module specifier for CommonJS as we execute all of the code up to require statement before we go to the next module
      - require(`${path}/counter.js`).count;
    - ESModules build up the whole module graph before any evaluation so we can’t have variables in module specifiers
      - can’t do import { count } from `${path}/counter.js`
      - can do dynamic imports - import(`${path}/foo.js`) - any file loaded this way is handled as the entry point to a separate graph and starts a new graph that is processed separately
      - loader caches module instances so only will be one module instance if a module appears in both graphs (modules fetched only once) - uses a module map for the cache; loader fetches URL and puts URL in module map
  - In node can have .mjs extension to signal a parse goal
- Instantiation - find boxes in memory to place all of the exported values but don’t fill them in yet; make imports/exports point to those boxes in memory called linking
  - depth first post-order traversal; goes down to bottom of graph to dependencies at bottom without other deps and sets up their exports
  - vs. CommonJS the entire export object is copied on export and any values are exported as copies
  - ES modules has live bindings = both modules point to same location in memory; when exporting module changes a value, the change will show up in the importing module; importing modules cannot change the values of their imports
  - live bindings help with wiring up all of the modules without running any code and helps with cyclic dependencies
  - instances and memory locations for exported/imported variables wired up
- Evaluation - run the code to fill in the boxes with variables’ actual values
  - executes the top-level code outside of functions to fill boxes in memory
  - evaluate module once and why there is a module map to cache the module by canonical URL so there is only one module record for each module
  - depth first post-order traversal
  - in CommonJS cyclic dependencies may lead to undefineds being imported/exported but in ES Modules with timeouts or eventual evaluation the values will not be undefined and update accordingly
- More async compared to CommonJS modules in Nodejs - loaded, instantiated and evaluated all at once
- Separate loader based on HTML spec

https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
https://kentcdodds.com/blog/super-simple-start-to-es-modules-in-the-browser
