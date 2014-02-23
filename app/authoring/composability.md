---
layout: default
---

# Composability

Yeoman offers multiple ways for generators to build upon common ground. There's no sense in always rewriting the same functionality, that's why we provide an API to use other generators inside your own.

In Yeoman, composability can be initiated in two ways. A generator can decide to compose itself with anoter (e.g.: `generator-backbone` uses `generator-mocha`). A end user may also initiate the composition (e.g.: Simon want to generate a backbone project with SASS and Rails).

**Note:** User composability feature haven't yet landed in the core. It is on our timeline and will land sooner than later!

## `generator.composeWith()`

The `composeWith` method allows your generator to declare that it wants to run side-by-side with another generator. The idea is that instead of implementing all of the functionality you want to offer yourself, if another generator offers it, you can use it to complete the set of features you want to scaffold.

### API

`composeWith` takes three parameter.

1. `namespace` - A String declaring the namespace of the generator you wish to compose with. By default, this namespace will be matched against available generators on the end user machine. This means it is a good idea to declare other generators you compose with as a [`peerDependencies`](http://blog.nodejs.org/2013/02/07/peer-dependencies/). We'll talk more about dependency type later on.
2. `options` - Options is an object containing an `options` and/or an `arguments` property. These will be passed to the called generator once it is called.
3. `settings` - A key/value object of internal settings.
    * `settings.local` allow you to define a path (String) to the requested generator. This allows you to reference one of your own sub-generator or to depend on specific versions of a generator. To do this, you'd usually declare the external generator as a simple [`dependencies` inside `package.json`](https://www.npmjs.org/doc/files/package.json.html#dependencies).
    * `settings.link` take one of two value (`weak` by default, or `strong`).
    
      The difference between those type of link is simply. A `weak` link won't run when the composability is initiated by the user. A `strong` link will always run.

      You should use `weak` link for features unrelated to the core of your generator (backend framework, CSS preprocessor). `strong` link should be use when requiring an action to occur. For example, scaffolding a _module_ by composing a _route_ and a _model_ generators.


When composing with a `peerDependencies`:

```js
this.composeWith('backbone:route', { options: {
  rjs: true
}});
```

When composing with a `dependencies`:

```js
this.composeWith('backbone:route', {}, {
  local: require.resolve('generator-bootstrap')
});
```

`require.resolve()` return the path from where Node.js would load the provided module.

## `generator.hookFor()`

A hook is very similar to `composeWith()` functionality. The main difference is that hooks are overridable by the end user.

When creating a hook, you give it a name; for example `test-framework`. With this name, the user can specify a custom generator to run as a command line option. For example, the user would run `yo generator --test-framework jasmine` to compose with the `generator-jasmine`. If the user provide no option, then a default generator is run.

As `hookFor` define new command line options, it can **only be run inside the constructor** method.

The API looks like this:

```js
this.hookFor('test-framework', {
  as: 'mocha' // In this case, `mocha` will be the default
});
```

## dependencies or peerDependencies

npm allow you to define multiple types of dependencies.

`dependencies` are installed locally to your generator. It is the best option when you need to control the version being run. Always prefer this option when possible.

`peerDependencies` on the other hand are installed one level higher than your generator. This mean if `generator-backbone` declared `generator-gruntfile` as a peer dependency, the folder tree would look this way:

```
├───generator-backbone/
└───generator-gruntfile/
```

When using `peerDependencies`, you need to be aware the module you request may be required by other module. This mean you need to be very careful not to create version conflict by requesting a specific version (or a narrow range of versions). Our recommendation when using `peerDependencies` is to always request _higher or equal to_ or _any_ available versions. For example:

```json
{
  "peerDependencies": {
    "generator-gruntfile": "*",
    "generator-bootstrap": ">=1.0.0"
  }
}
```
