# npm3 - #10999 - Npm v3 non-determinism does actually result in different code (not just tree structure)

Original bug can be found here: #10999.
With npm 3, for the first time other packages can influence each other's dependencies.

## Up and Running

```
$ npm install test-b --save
$ npm install test-a --save
$ node index.js
```

This will print:

```
1.0.0
1.0.1
```

Now let's try the opposite order:

```
$ rm -rf node_modules # just restore things to their previous state
$ git checkout package.json # just restore things to their previous state
$ npm install test-a --save
$ npm install test-b --save
$ node index.js
```

This will print:

```
1.0.0
```

# Explanation

As you can see, install order actually affects the bits that will be run. The key here is that test-a has an *absolute dependency* on test-c 1.0.0, while test-b has a semver range dependency on test-c ^1.0.0. That means if test-b is installed first, it correctly picks up 1.0.1, then a will of course want 1.0.0. However, if a installs first, then 1.0.0 will be installed, which satisfies b, and thus both get 1.0.0.

The important thing here is that anyone *ELSE* that has an absolute dependency (and is alphabetically before you), effectively sabotages YOUR package. The only remedy is then on the *end user* to manually fiddle with thier shrinkwrap. As a package author, you can't actually do anything, other than explain to your users the subtle effects *other packages* have on your package, then try to walk them through their each time unique shrinkwrap solution.