---
layout: post
title:  "Transpiling And Publishing ES9 NPM Modules With Babel 7"
date:   2018-11-17
---

<span class="dropcap">W</span>ith 2018 coming to an end, we have seen adoption of the Node.js 8 runtime by [AWS Lambda][aws-lambda] as well as [Google Cloud Functions][google_cloud_functions]. Native support for ES8's async functions using async/await has been much anticipated, and is definitely a much welcomed step forward for anyone using server-side Javascript.

However, with the release of ES2018 (ES9) there are now even more cool features to add to your developer toolkit, such as [Async Iteraters][async-iterators] and [Object Rest/Spread Properties][rest-elements]. Unfortunately support for these features requires Node v10+. While many developers may be afforded the luxury of being able to use whichever node version they desire, for others (e.g. those using AWS Lambda) the requirements can be more stringent.

Imagine you are a developer and you have a module that you want to share with the world via NPM. You love using the latest and greatest Javascript, so naturally your package contains some ES9 features. How do you ensure that your NPM package can be used by as wide an audience as possible?

The answer: Babel.

Babel has caught quite a bit of flack in recent years, and in some cases, for good reason. Debugging can become more difficult with Babel, and some of its features (e.g. minification) are generally not beneficial server-side. However, new releases for the JS language are going to continue to come down the pipeline. In my opinion, establishing a workflow for your project that allows you to take advantage of new features without forcing your consumers to upgrade Node is a good thing.

Fortunately, setting up a workflow to transpile ES9 code using Babel 7 is fairly straightforward. The following steps will get you started with the skeleton of a project that will allow you to create backwards compatible NPM packages using ES9 code:

Create your project folder, and set up NPM. 
{% highlight bash %}
mkdir myProject
cd myProject
npm init
{% endhighlight %}


Now install babel:

{% highlight bash %}
npm install --save-dev @babel/cli @babel/core @babel/preset-env @babel/register
{% endhighlight %}

Configure Babel by creating a `.babelrc` file in your root directory with the following:
{% highlight json %}
{
  "presets": [
    [
      "@babel/preset-env", {
        "targets": {
          "node": "current"
        }
      }
    ]
  ]
}
{% endhighlight %}

Within your project folder, create a `src` and `lib` folder:
{% highlight bash %}
mkdir lib src
{% endhighlight %}

The `src` folder will contain your ES9 code, and will be what you would keep in your version control (e.g. Git). The `lib` directory will contain your transpiled code, and will be what you upload to NPM.

Setup your build script to output to your `lib` directory:
{% highlight json %}
"scripts": {
    ...
    "build": "./node_modules/.bin/babel src --out-dir lib",
    ...
  },
{% endhighlight %}

NPM also supports a [prepare][npm-prepare] script that will be run before the package is packed and published. You can use this to ensure that your module is transpiled when you go to pubslish:

{% highlight json %}
"scripts": {
    ...
    "prepare": "npm run build",
    ...
  },
{% endhighlight %}

Update your module's entrypoint within the `package.json` to point to your `lib` directory. For example:

{% highlight json %}
// package.json
{
  "name": "es9-test",
  "main": "lib/index.js",
  ...
}
{% endhighlight %}

You will want to be able to run tests on your code as well (hopefully!). If you are using Mocha your test script will look something like this:

{% highlight json %}
"scripts": {
    "test": "mocha --require @babel/register test/*.test.js",
    ...
  },
{% endhighlight %}

The key portion of this being the inclusion of [@babel/register][babel-register] via the require hook.

To prevent the non-transpiled ES9 code from being uploaded to NPM you can also add an `.npmignore` file that excludes the `/src/` directory. Similarly, you may want to add the `lib` directory to your `gitignore` to exclude your transpiled code from being checked into version control.

Your `package.json` should now look something like the following:


{% highlight json %}
{
  "name": "...",
  "version": "1.0.0",
  "description": "...",
  "main": "lib/index.js",
  "scripts": {
    "test": "mocha --require @babel/register test/*.test.js",
    "build": "./node_modules/.bin/babel src --out-dir lib",
    "prepare": "npm run build"
  },
  "dependencies": {
   ...
  },
  "devDependencies": {
    "@babel/cli": "^7.1.5",
    "@babel/core": "^7.1.5",
    "@babel/preset-env": "^7.1.5",
    "@babel/register": "^7.0.0",
    "mocha": "^5.2.0",
  }
}
{% endhighlight %}

Now you are ready to publish a backwards compatible NPM module written with ES9 features.

[jekyll-gh]: https://github.com/mojombo/jekyll
[jekyll]:    http://jekyllrb.com
[google_cloud_functions]: https://cloud.google.com/functions/docs/concepts/nodejs-8-runtime
[aws-lambda]: https://aws.amazon.com/blogs/compute/node-js-8-10-runtime-now-available-in-aws-lambda/
[async-iterators]: https://github.com/tc39/proposal-async-iteration
[rest-elements]: https://github.com/tc39/proposal-object-rest-spread
[babel-register]: https://babeljs.io/docs/en/babel-register
[npm-prepare]: https://docs.npmjs.com/misc/scripts
