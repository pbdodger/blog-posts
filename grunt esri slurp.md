Running the [Dojo Build System](http://dojotoolkit.org/reference-guide/build/) with [ESRI's JavaScript API for ArcGIS Server](http://js.arcgis.com) has been a problem that we have been trying to tackle for a [very](http://gis.utah.gov/speed-up-your-esri-javascript-api-webapp/), [very](http://gis.utah.gov/esri-jsapi-3-4-and-the-dojo-build-system/), [very](http://gis.utah.gov/the-esri-api-for-javascriptdojo-build-system-saga-continues/) long time. This post is about one of the key components to our current strategy that allows us to utilize a continuous integration server and obtain that mythical single JavaScript file build. 

When version 3.4 of the ESRI JS API was released, as ESRI employee informed us of an undocumented AMD build. It was available on the CDN just like the other builds, [http://js.arcgis.com/3.10amd/init.js](http://js.arcgis.com/3.10amd/init.js), but it had one key difference: it contained no [layer files](http://dojotoolkit.org/reference-guide/build/profiles.html#id6). It is straight up AMD modules that have been optimized by the dojo build system. This simplified working with the build system _immensely_. With a bit of massaging you can take these modules and create an ESRI [package](http://dojotoolkit.org/reference-guide/loader/amd.html#id6) that can be ingested by Dojo's build system with relatively ease. This _"bit of massaging"_ is made trivial with the use of [`grunt-esri-slurp`](https://github.com/steveoh/grunt-esri-slurp)

There are a few problems that this [grunt](http://gruntjs.com/) plugin solves. The first is that the AMD build is intended to be used from a CDN and the Dojo build system expects the packages to exist on disk. `grunt-esri-slurp`, given a [list of modules](https://github.com/steveoh/grunt-esri-slurp/blob/master/tasks/esriModules-3.10.js), will download each resource one at a time from the CDN to the package location for your project. `grunt-esri-slurp` [builds the module list](https://github.com/steveoh/grunt-esri-slurp/blob/master/tasks/esriModuleBuilder.js) and stores the list for each api version that it supports ([3.8+](https://github.com/steveoh/grunt-esri-slurp/issues/1)). 

The second problem with the vanilla AMD build is that the modules have already been optimized by the dojo build system. These optimizations render the modules incompatible for use with the build system again. For example, the build system concatenates all of an AMD modules dependencies into a single string with a `.split(',')` on the end like this:
```
define('dep1,dep2,dep3'.split(','), function(s,G,H){...});
``` 
If you tried to use this vanilla AMD module within the build system it will not be able to parse the dependencies and the build will fail. `grunt-esri-slurp` [unwinds](https://github.com/steveoh/grunt-esri-slurp/blob/master/tasks/unwinder.js) the dependencies back into an array of strings like this:
```
define(['dep1','dep2','dep3'], function(s,G,H){...});
```

Until now we have been focusing solely on javascript. Optimizing CSS can save trips to your servers and time on your first page load. The dojo build system will optimize all of the CSS found in your packages. Our strategy is to create [one top level CSS](https://github.com/agrc/AGRCJavaScriptProjectBoilerPlate/blob/master/src/app/resources/App.css) for our websites and `@import` the dependencies. The dojo build system will then concatentate these dependencies into the top level CSS and your website will only make one request. 

The ESRI AMD build package contains a different file structure than the best practice where the entire dojo framework is nested inside a folder called dojo. `grunt-esri-slurp` [unwinds](https://github.com/steveoh/grunt-esri-slurp/blob/master/tasks/unwinder.js) the CSS file paths to make the `dojo`, `dijit`, and `dojox` packages at the same level as ESRI and your own packages.

How to get started using

boilerplate, https://github.com/tomwayson/esri-slurp-example

Should we break this up into multiple posts?


The original solution to this problems was a [bash script](https://github.com/agrc/AGRCJavaScriptProjectBoilerPlate/blob/30782f918d883dd67d99b3d966f7501817f1a234/slurp_esri_modules.sh). However, this proved difficult for cross-platform support and was hard to maintain. Earlier this year, [steveoh](https://github.com/steveoh) ported the bash script to a grunt task, [`grunt-esri-slurp`](https://github.com/steveoh/grunt-esri-slurp). Since then we have worked to make it much better than the original bash script.