# About programming with Node.JS
Using Javascript and JQuery limited to client-side programming, I have been interested in the Node.JS project for a while. Wouldn't it be great to use Javascript everywhere and not having to integrate different programming languages to make something work smoothly?

I have been using PHP as my main programming language until recently, but have left that path for good. Despite PHP version 7 being rock-solid, there are some things I don't like about the language, such as having to select the features you will be using in your project. With the modular system of Node.JS this works quite differently, basically picking and choosing packages that are readily out there. You just require them (using CommonJS terminology) and that's it.

One such very popular package is ExpressJS, making it easy for you to build your own CMS very easily. What I do not like about integrating these packages, is that you get a bunch of other packages installed along, some of which seem to serve no purpose at all. True, sometimes the developers do a cleanup and remove any stale packages they required, but still.

Also, rather than depending on a ready-made MVC, I wanted to understand exactly how I could build one from scratch. That's what I did in PHP7 at first, and it worked quite well, with routes you can define in configuration files (JSON), and data coming from different sources (MySQL, MongoDB, XML, and even JSON). It sparked my interest, and enthousiasm to rewrite the whole MVC in Javascript and port it to Node.JS, where JSON and MongoDB is common good.

Although far from done with my personal project, I wanted to write something about my experiences so far.

## CommonJS vs ECMAScript6
One of the things I have noticed, is that most packages out there are still written in CommonJS, and expecting others to write in CommonJS as well. This means that these modules are not taking advantage of all that ECMAScript6 has to offer. For those who don't know, ECMAScript6 (also known as ES6) defines a newer syntax compared to CommonJS. Speaking of modules, the main difference is that CommonJS actually executes the code being required. ES6 only references the code using imports, but does not (entirely) execute it. This means that stale, unused modules will not affect the application as much.

I'm also not a fan of using technology that is going to be "old" pretty soon. It means that the code is no longer being developed any further and will, at some time in the future, lose support. So, writing the MVC, I soon discovered that I preferred supporting ECMAScript6 and leaving CommonJS behind me.

This also meant that I had to adapt pretty much every module to use the new way of exporting and importing modules.

## Autoloader
Because my MVC project uses a lot of classes that all interact, I wanted to find a way to automatically load all the modules I need, rather than specifying in each module the ones I would need because I might have classes depending on these. Pretty soon, I would end up doing the same as existing packages, importing modules just because at some point in time, I needed them. Then forgetting they were there.

I actually had a pretty neat module autoloader running in CommonJS, but had to rewrite and rethink completely the way I did that when working with ES6 modules. Looking at how others would do that, I came out empty-handed. Most autoloaders seem to have been written for CommonJS, not ES6. Now, if there is a challenge, I am glad to take it on.

The result is the Namespace module, which is part of the Core package of the CMS. It basically scans a directory for ES6 modules and registers these. Then, once you actually need the module, you can load it into the namespace. I have made sure that I can also load the same module as a global, so that I do not constantly need to reference a class using the namespace, although this is possible. Even more, all modules are automatically registered when importing the Core package.

I did not do the effort of also automatically load them, as the Core modules are just part of a bigger project with more modules being developed in other packages. More about the Core package is covered in [[this post]].

The Namespace module is described in more detail in yet another blog post, but to give you some heads-up, this is how I import (and export) the Core package from within the Framework package:

```
import { dirname } from 'path';
import { fileURLToPath } from 'url';

import Namespace from '@cubo-cms/core';

const basePath = dirname(fileURLToPath(import.meta.url));

Namespace.autoRegister('./lib', basePath);

export default Namespace;
```

It's pretty self-explanatory, right? I just import the Core package. Remember I said, it will automatically register all modules in that package, so they are already available to load into the namespace at that point. Before doing that, I register the modules from the Framework package by using `Namespace.autoRegister(sourceFolder, basePath)`. And I export the same Namespace again to be available for the CMS.

If I would like to actually load the modules, so that they become available through the namespace, or as globals, and then simply run my application, I would use:

```
import Namespace from '@cubo-cms/framework';
await Namespace.autoLoad();

let app = new Application('#/application.json');

app.run();
```

The `await Namespace.autoLoad()` does the magic here. Since all functions are asynchronous, we'll have to *await* its completion, before we can actually use the loaded modules. The module **Application** is one of these modules, the main one actually, that is imported from the Framework package.

To understand more about the workings of the Namespace module in particular, please read [[this post]].

## Clean programming
Now what I like so much about the way I do things is that I do not depend so much on code others have written. For one, I'd like the challenge of working things out myself rather than depending on others. Secondly, I do not need to be afraid that things will break if other programmers change something, or leave the development of the code for what it is. Lastly, look at how clean my *node_modules* folder is:

[[Include image]]

This is because I code everything myself. Of course, when using MongoDB or MySQL, I start installing those dependencies, and adding a bus-load of modules with it. It is not my intention to completely rewrite drivers for those solutions. But the Core and Framework packages do not include those. Data retrieval, up to the Framework, is completely done through JSON.

## Conclusion
Writing in ECMAScript has so far been a quest with trials and errors, but with a great learning curve. I cannot wait until I have my MVC up and running, and can develop my CMS on top of it. I am sure to let you know when I am done.
