# Effective Esy Packaging:

## What Is Esy:

`esy` is a package manager command line utility that provides a fast, sandboxed
workflow for native/compiled languages. It makes native compilation, well -
easy. There is one main command you ever need to use in `esy`, and that one
command is.. `esy`. You just run `esy` from your project root and `esy` figures
out what needs to happen to fetch dependencies and build them and build your
project.

```
npm install -g esy
```

## What Is A Package:

A package is a versioned collection of code shared on `npm` or `opam`. `esy`
package names should consist of lower case, hyphens or underscores.  Here's how
we will illustrate a package named `my-package`:

      ┌────────────╮
      │my-package  │
      │            └─────────────────────────────────────────┐
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      │                                                      │
      └──────────────────────────────────────────────────────┘

The only requirement of an `esy` package, is that it contains a
`package.json`/`esy.json` file describing the package's name and package
dependencies(just like with `npm` packages). It will also contain unbuilt
source files.

Configuring packages can be boring, but there's a tool called
[`pesy`](https://github.com/jordwalke/pesy)`pesy` which makes it more fun.
Let's use that in these examples to quickly setup/configure examples.

```sh
npm install -g pesy
mkdir my-package && cd my-package
pesy
```

`pesy` has created a starter package for us called `my-package`.  At this
point, it's just a regular `esy` project and you _could_ forget about `pesy`.

As always you can just run `esy` in the project directory and it does
everything for you to build the project/dependencies.

```
┌────────────╮
│my-package  │
│            └───────────────────────────────────────────┐
│  ./                                ./library/          │
│  ┌────────────────────────┐ ┌──────────┐ ┌──────────┐  │
│  │package.json            │ │Util.re   │ │Test.re   │  │
│  │                        │ │          │ │          │  │
│  │{                       │ │          │ │          │  │
│  │ "name": "my-package",  │ │          │ │          │  │
│  │ "version": 1.0.0       │ │          │ │          │  │
│  │ "dependencies": {      │ │          │ │          │  │
│  │  "@opam/dune": "*",    │ │          │ │          │  │
│  │   ...                  │ │          │ │          │  │
│  │ }                      │ │          │ │          │  │
│  │}                       │ │          │ │          │  │
│  └────────────────────────┘ └──────────┘ └──────────┘  │
│                                                        │
└────────────────────────────────────────────────────────┘
```


### Dependencies Between Packages:

Packages express which other packages they depend on inside their
`package.json`'s `"dependencies"` field. `esy add` will help you add new
dependencies, and will update your json file for you. Run `esy add` followed by
to add the `@reason-native/console` package as a dependency, then run `esy` to
refetch/rebuild whatever is necessary (as always).

```sh
# Adds this to your package.json
esy add @reason-native/console
# Fetches and rebuilds whatever is necessary.
esy
```

We'll represent dependencies using the double arrow `═══>`

```
┌──────────────╮                                            ┌──────────────────────╮
│my-package    │                                            │@reason-native/console│
│              └───────────────────────────────────────┐    │                      └───────────────┐
│                                                      ╞═══>│                                      │
│ ┌──────────────────────────────┐ ┌───────┐ ┌───────┐ │    │ ┌───────────────────────────────┐    │
│ │package.json                  │ │Util.re│ │Test.re│ │    │ │package.json                   │    │
│ │                              │ │       │ │       │ │    │ │                               │    │
│ │{                             │ │       │ │       │ │    │ │{                              │... │
│ │ "name": "my-package",        │ │       │ │       │ │    │ │"name":"@reason-native/console"│    │
│ │ "version": 1.0,              │ │       │ │       │ │    │ │"version": 2.0.0               │    │
│ │ "dependencies": {            │ │       │ │       │ │    │ │...                            │    │
│ │  "your-package":"2.0.0"      │ │       │ │       │ │    │ └───────────────────────────────┘    │
│ │  "@reason-native/console":"*"│ │       │ │       │ │    │                                      │
│ │ }                            │ │       │ │       │ │    │                                      │
│ │}                             │ │       │ │       │ │    │                                      │
│ └──────────────────────────────┘ └───────┘ └───────┘ │    │                                      │
│                                                      │    │                                      │
└──────────────────────────────────────────────────────┘    └──────────────────────────────────────┘
```

> Lock Files: Running `esy` also creats an `esy.lock` directory (the lock
> directory is just a replayable log file recording what just happened on the
> network when installing - more on that later).

### Build:

Esy expects your `package.json` to include an `"esy": {  }` section. That
section should contain a `"build"` field, which specifies the command to build
the package. For example it could be `"make"`. This project happens to use a
build system called `dune` instead of `make`.  `esy` will invoke your `build`
field command whenever it is necessary to rebuild your project - you don't need
to think about when then will happen - just always run `esy` from the project
root as always
Esy keeps track of your packages' build results, as well as all your
dependencies' build results in a global build cache that you never need to
think about. But those build results do exist somewhere, and we will represent
them in the diagrams under the dotted line, for each pacakge.

```
┌──────────────╮                                            ┌──────────────────────╮
│my-package    │                                            │@reason-native/console│
│              └───────────────────────────────────────┐    │                      └───────────────┐
│                                                      ╞═══>│                                      │
│ ┌──────────────────────────────┐ ┌───────┐ ┌───────┐ │    │ ┌───────────────────────────────┐    │
│ │package.json                  │ │Util.re│ │Test.re│ │    │ │package.json                   │    │
│ │                              │ │       │ │       │ │    │ │                               │    │
│ │{                             │ │       │ │       │ │    │ │{                              │... │
│ │ "name": "my-package",        │ │       │ │       │ │    │ │"name":"@reason-native/console"│    │
│ │ "version": 1.0,              │ │       │ │       │ │    │ │"version": 2.0.0               │    │
│ │ "dependencies": {            │ │       │ │       │ │    │ │...                            │    │
│ │  "your-package":"2.0.0"      │ │       │ │       │ │    │ └───────────────────────────────┘    │
│ │  "@reason-native/console":"*"│ │       │ │       │ │    │                                      │
│ │ }                            │ │       │ │       │ │    │                                      │
│ │}                             │ │       │ │       │ │    │                                      │
│ └──────────────────────────────┘ └───────┘ └───────┘ │    │                                      │
│                                                      │    │                                      │
│  --------------------------------------------------  │    │ ------------------------------------ │
│                   build results                      │    │              build results           │
└──────────────────────────────────────────────────────┘    └──────────────────────────────────────┘
```


## Libraries:

The most common kind of "build result" is a built "library". Libraries are like
subpackages. They are named groupings of compiled source files.

> Example:
> `my-package`, might set its `"build"` field to `"make"`, which compiles
> `my-package` into two libraries (subpackages) `my-package.lib` and
> `my-package.test`.  `my-package.lib` might include `Util.re` and
> `my-package.test` might include `Other.re`. In practice no one uses `make` to
> build libraries, but you could.
>
> ```json
>  {
>   "name": "my-package",
>   "esy": {
>     "build": "make"
>   },
>   ...
> }
> ```

Each library `package-name.xyz` gets to decide its module "namespace". The
module namespace is the "Reason" module name that all the `.re` files will be
accessed through in actual code. Often the namespace is just the capitalized
camel case form of the package name.

For example, our dependency `@reason-native/console` has a library named
`console.lib` whose namespace is `Console`. That means that when our package
depends on `@reason-native/console`, we can use its `console.lib` library, to
access its `ObjectPrinter.re` like: `Console.ObjectPrinter`.

Here's a diagram of our package and our dependency, each with their respective
libraries "subpackages" shown under the dotted line in the build results
section.

```
┌──────────────╮                                            ┌──────────────────────╮
│my-package    │                                            │@reason-native/console│
│              └───────────────────────────────────────┐    │                      └───────────────┐
│                                                      ╞═══>│                                      │
│ ┌──────────────────────────────┐ ┌───────┐ ┌───────┐ │    │ ┌───────────────────────────────┐    │
│ │package.json                  │ │Util.re│ │Test.re│ │    │ │package.json                   │    │
│ │                              │ │       │ │       │ │    │ │                               │    │
│ │{                             │ │       │ │       │ │    │ │{                              │... │
│ │ "name": "my-package",        │ │       │ │       │ │    │ │"name":"@reason-native/console"│    │
│ │ "version": 1.0,              │ │       │ │       │ │    │ │"version": 2.0.0               │    │
│ │ "dependencies": {            │ │       │ │       │ │    │ │...                            │    │
│ │  "your-package":"2.0.0"      │ │       │ │       │ │    │ └───────────────────────────────┘    │
│ │  "@reason-native/console":"*"│ │       │ │       │ │    │                                      │
│ │ }                            │ │       │ │       │ │    │                                      │
│ │}                             │ │       │ │       │ │    │                                      │
│ └──────────────────────────────┘ └───────┘ └───────┘ │    │                                      │
│                                                      │    │                                      │
│  --------------------------------------------------  │    │ ------------------------------------ │
│                             ┌─────────────────────┐  │    │            ┌────────────────────┐    │
│                             │my-package.lib       │  │    │            │console.lib         │    │
│                             ├─────────────────────┤  │    │            ├────────────────────┤    │
│                             │namespace: MyPackage │  │    │            │namespace: Console  │    │
│                             └─────────────────────┘  │    │            └────────────────────┘    │
└──────────────────────────────────────────────────────┘    └──────────────────────────────────────┘

```

### Building Libraries:

We haven't mentioned exactly how these libraries get built, only _that_ they
are built when `esy` invokes your `"build"` field. Building these libraries is
the responsibility of your build system (`make`/`dune` etc). To configure how
your libraries are built, and what their namespaces are, you would consult your
build system documentation.

If you created your project up with `pesy`, then you can use `pesy` to
configure and update your `dune` library build configuration just by editing
your `package.json`.

Edit your `package.json`'s `"buildDirs"` section to add/name/namespace
libraries then run:

```
esy pesy
esy
```
`esy pesy` will reconfigure your libraries/namespaces based on your `buildDirs`
section then `esy` will build whatever it needs to (as always). Imagine
`esy pesy` just updating your `Makefile` for you - you only need to run it if
you've reconfigured some library that needs to update your build config.
Ideally this would be automated, and you could always hand-edit the dune
config instead.



### Libraries Depending On Libraries

We already saw that packages can depend on packages (remember when we called
`esy add`). But if you were to write `Console.log("hi")` inside of your
package's `Util.re`, it won't compile, because your `Util.re` (which is inside
of your library `my-package.lib`) needs to depend on `@reason-native/console`'s
library named `console.lib`.

`pesy` can help us do this too. Edit the `buildDirs` section for
`my-package.lib` and add `"require": [ "console.lib" ]` to its section. Then
run:

```
esy pesy   # Just to update library config
esy        # To build whatever needs to be built.
```

```
┌──────────────╮                                            ┌──────────────────────╮
│my-package    │                                            │@reason-native/console│
│              └───────────────────────────────────────┐    │                      └───────────────┐
│                                                      ╞═══>│                                      │
│ ┌──────────────────────────────┐ ┌───────┐ ┌───────┐ │    │ ┌───────────────────────────────┐    │
│ │package.json                  │ │Util.re│ │Test.re│ │    │ │package.json                   │    │
│ │                              │ │       │ │       │ │    │ │                               │    │
│ │{                             │ │       │ │       │ │    │ │{                              │... │
│ │ "name": "my-package",        │ │       │ │       │ │    │ │"name":"@reason-native/console"│    │
│ │ "version": 1.0,              │ │       │ │       │ │    │ │"version": 2.0.0               │    │
│ │ "dependencies": {            │ │       │ │       │ │    │ │...                            │    │
│ │  "your-package":"2.0.0"      │ │       │ │       │ │    │ └───────────────────────────────┘    │
│ │  "@reason-native/console":"*"│ │       │ │       │ │    │                                      │
│ │ }                            │ │       │ │       │ │    │                                      │
│ │}                             │ │       │ │       │ │    │                                      │
│ └──────────────────────────────┘ └───────┘ └───────┘ │    │                                      │
│                                                      │    │                                      │
│  --------------------------------------------------  │    │ ------------------------------------ │
│                             ┌─────────────────────┐  │    │            ┌────────────────────┐    │
│                             │my-package.lib       │  ╞════════════════>│console.lib         │    │
│                             ├─────────────────────┤  │    │            ├────────────────────┤    │
│                             │namespace: MyPackage │  │    │            │namespace: Console  │    │
│                             └─────────────────────┘  │    │            └────────────────────┘    │
└──────────────────────────────────────────────────────┘    └──────────────────────────────────────┘
```

The `require` section updated our library's config to make sure our
`my-package.lib` depends on `console.lib`. (See the new double arrow).

Now `Util.re` can include a `Console.log("hi")` call and will compile correctly!


#### Libraries Recap:

A typical conversation about esy packages and libraries might go something like
this:

> - John: "Hey I heard you released an awesome package. What's the package name?"
> - Sue : "Oh, it's named `package-name`. You can depend on it by doing `esy add package-name`."
> - John: "Cool, but how do I actually use the stuff inside? What libraries does it include?"
> - Sue : "Oh, it includes one library named `package-name.lib`. After you add `package-name` as a package dependency, configure your build to use the library named `package-name.lib`.
> - John: "Okay, and then how do I access the modules inside of `package-name.lib` once I've done that?
> - Sue : "Through that library's namespace of course!"
> - John: "Yeah yeah, but what is the actual string you used for the namespace?"
> - Sue : "Oh I just took the package name and turned it into an upper camel case '`PackageName`'"
> - John: "Okay so if I depend on your package `package-name`, and configure my build to use your library `package-name.lib`, then I'll be able to write `PackageName.Utils.foo()`?
> - Sue: "You got it!"

#### Libraries Recommended Convention:

By convention, an esy package `my-package` that you publish to npm should
usually only build _one_ library named `my-package.lib`, which has a namespace
of `PackageName`(the upper camel cased package name). There are exceptions to
this rule.


## Executables:

Each esy package can build more than just libraries. It can also build
executables. This means that consumers of your package can not only use your
built code namespaces but they can also use your built executables for whatever
purposes they like. These executables are built from sources just like your
libraries were built from source. What makes esy special is its ability to
build things from sources in a really predictable and fast way, whether that be
libraries or executables.

```
┌──────────────╮
│my-package    │
│              └───────────────────────────────────────┐
│                                                      │
│ ┌──────────────────────────────┐ ┌───────┐ ┌───────┐ │
│ │package.json                  │ │Util.re│ │Test.re│ │
│ │                              │ │       │ │       │ │
│ │{                             │ │       │ │       │ │
│ │ "name": "my-package",        │ │       │ │       │ │
│ │ "version": 1.0,              │ │       │ │       │ │
│ │ "dependencies": {            │ │       │ │       │ │
│ │  "your-package":"2.0.0"      │ │       │ │       │ │
│ │  "@reason-native/console":"*"│ │       │ │       │ │
│ │ }                            │ │       │ │       │ │
│ │}                             │ │       │ │       │ │
│ └──────────────────────────────┘ └───────┘ └───────┘ │
│                                                      │
│  --------------------------------------------------  │
│   ┌─────────────────────┐  ┌─────────────────────┐   │
│   │MyExecutable.exe     │  │my-package.lib       │   │
│   ├─────────────────────┤  ├─────────────────────┤   │
│   │111001001010111000101│  │namespace: MyPackage │   │
│   └─────────────────────┘  └─────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

TODO: Add more examples with other build systems.

## Package Versioning: (TODO)

- Quick Introduction To Semantic Versioning.
- `package.json` `dependencies`
- Constraint solving.

**SemVer is Not a Silver Bullet - And That's Okay:**
- Dependencies can misuse semantic versioning.
- lockfiles provide the stability that semantic versioning could never fully achieve.
- resolutions compensate for the reality that package authors may not always
provide perfect constraints and even perfect constraints might be overly
constrained.

**Smaller Packages May Reduce Version Conflicts:**

## Suggested Reason Version Conventions
- What is a breaking change.
- What is not a breaking change? Very few things. But that's harmful if true.
  Let's remove some programming patterns that allow more things to be
  non-breaking.
  - Limiting the use of `open`.
  - Structuring your modules in a certain way.
    - Document which modules may be opened without breaking your code.
    - Create a `Package.Types` module that people can open just to get
      variants and record labels in scope.
  - Most importantly, every package should specify in their documentation which
    usage is guaranteed to not break including which modules may be opened
    without conflicting with consumers' code.

## Developing In Large Closed Source Code-Bases: (TODO)
========================================



