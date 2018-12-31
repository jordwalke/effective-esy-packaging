Effective Esy Packaging:
========================================

### What Is A Package:

A package is a versioned collection of code shared on `npm` or `opam`. `esy`
package names should consist of lower case, hyphens or underscores.  Here's how
we will illustrate a package named `my-package`:

      ┌────────────┐
      │my-package  │
      │            └────────────────────────────────────────┐
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      │                                                     │
      └─────────────────────────────────────────────────────┘


The only requirement of an `esy` package, is that it contains a
`package.json`/`esy.json` file describing the package's name and package
dependencies(just like with `npm` packages). It will also contain unbuilt
source files.


      ┌────────────┐
      │my-package  │
      │            └────────────────────────────────────────┐
      │  ┌─────────────────────┐   ┌─────────┐ ┌─────────┐  │
      │  │package.json         │   │MyMod.re │ │MyMod2.re│  │
      │  │                     │   │         │ │         │  │
      │  │{                    │   │         │ │         │  │
      │  │ "name": "my-package"│   │         │ │         │  │
      │  │ "version": 1.0      │   │         │ │         │  │
      │  │ ...                 │   │         │ │         │  │
      │  │}                    │   │         │ │         │  │
      │  └─────────────────────┘   └─────────┘ └─────────┘  │
      └─────────────────────────────────────────────────────┘


### Building Libraries:

`esy` packages implement a field called `build` in their `package.json` that
`build` command's job is to takes the package source files and turn them into
"libraries". Libraries are just named groupings of compiled source files in
your package. It's easiest to think of them as "subpackages".

Each library has a library name and a "namespace".

- The library name must be prefixed with the package name, and contain only
  alphanumeric(or hyphens). For example, `package-name.lib`. The prefixing
  convention helps remind you that libraries are like "subpackages".
- The namespace is an upper cased word that consumers will access all the `.re`
  modules through. For example if you have a package `my-package`  with a
  library named `my-package.lib` that has a namespace of `NameSpace`, people
  will use that library to access `ModuleName.re` like: `NameSpace.ModuleName`.

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

By convention, an esy package `my-package` should build _one_ library named
`my-package.lib`, which has a namespace of `PackageName`(the upper camel cased
package name).

      ┌────────────┐
      │my-package  │
      │            └────────────────────────────────────────┐
      │  ┌─────────────────────┐   ┌─────────┐ ┌─────────┐  │
      │  │package.json         │   │MyMod.re │ │MyMod2.re│  │
      │  │                     │   │         │ │         │  │
      │  │{                    │   │         │ │         │  │
      │  │ "name": "my-package"│   │         │ │         │  │
      │  │ "version": 1.0      │   │         │ │         │  │
      │  │ ...                 │   │         │ │         │  │
      │  │}                    │   │         │ │         │  │
      │  └─────────────────────┘   └────┬────┘ └────┬────┘  │
      │                                 │           │       │
      │  ----------------------build----│-----------│-----  │
      │                                 v           v       │
      │                            ┌─────────────────────┐  │
      │                            │my-package.lib       │  │
      │                            ├─────────────────────┤  │
      │                            │namespace: MyPackage │  │
      │                            └─────────────────────┘  │
      │                                                     │
      └─────────────────────────────────────────────────────┘


One thing brushed over here was the way you configure your build system to
build a subset of `.re` files into a named "library" with a namespace.

If using [pesy](https://github.com/jordwalke/pesy), the
[configuring](https://github.com/jordwalke/pesy#configuring) section of the
docs explain how to do this.

Another thing brushed over here was how to configure your build system to
consume a library. There's a pesy section of the docs explaining how to [do
that](https://github.com/jordwalke/pesy#consuming-new-package-and-library-dependencies)

Each esy packages can build more than just libraries. It can also build
executables. This means that consumers of your package can not only use your
built code namespaces but they can also use your built executables for whatever
purposes they like. These executables are built from sources just like your
libraries were built from source. What makes esy special is its ability to
build things from sources in a really predictable and fast way.

      ┌────────────┐
      │my-package  │
      │            └────────────────────────────────────────┐
      │  ┌─────────────────────┐   ┌─────────┐ ┌─────────┐  │
      │  │package.json         │   │MyMod.re │ │MyMod2.re│  │
      │  │                     │   │         │ │         │  │
      │  │{                    │   │         │ │         │  │
      │  │ "name": "my-package"│   │         │ │         │  │
      │  │ "version": 1.0      │   │         │ │         │  │
      │  │ ...                 │   │         │ │         │  │
      │  │}                    │   │         │ │         │  │
      │  └─────────────────────┘   └────┬────┘ └────┬────┘  │
      │                                 │           │       │
      │  ----------------------build----│-----------│-----  │
      │                                 v           v       │
      │   ┌─────────────────────┐  ┌─────────────────────┐  │
      │   │MyExecutable.exe     │  │my-package.lib       │  │
      │   ├─────────────────────┤  ├─────────────────────┤  │
      │   │111001001010111000101│  │namespace: MyPackage │  │
      │   └─────────────────────┘  └─────────────────────┘  │
      │                                                     │
      └─────────────────────────────────────────────────────┘
      
TODO: Add more examples with other build systems.



**Next: Dependencies between libraries.**
