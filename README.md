# A Build Language

Notes on a procedural language for builds.

## What?

Language-specific tools like `cargo` will never be enough. Eventually you need
to create artifacts that have nothing to do with your progamming language.
Examples include container images, system packages, HTML documentation, or just
plain old text files.

## Prior Art

People have solved this problem before with the following

* shell scripts 
* Makefiles
* Systems that put a new spin on Makefiles (Rake, Mage)
* A "real" interpreted language (Python, Ruby)

## Requirements

Builds are an interesting problem. The ideal build is

* portable 
* fast  
* easy to understand
* easy to hack
* able to "shell out" to other programs, if necessary

### Portable

Portability means that the build tool itself doesn't have weird dependencies. It
also means that the tool will run on various common architectures. It might even
mean Windows support.

### Fast

We can make builds faster by

* caching things to avoid doing work we don't need to do
* doing _some_ things concurrently 

### Easy to Understand

Builds are written in some **language**, whether it's a scripting language like
bash or Python, a config language like YAML or TOML, or a custom DSL like sbt.

A language is easy to understand when it 

* is concise, but not _too_ concise.
* resembles a language you already know
* has good documentation

### Easy to Hack

Sometimes you just need to hack something into the build. 
A language is easy to hack when it

* is well-defined enough for static-analysis tools
* is procedural-ish, so the type system lets you move stuff around without complaining
* fails gracefully, with good error messages
* is readable by newcomers

From a language perspective, this means functions have side effects. Builds are
all about side effects: writing things to disk, for instance. But we don't want
to be as loosey goosey as bash, where almost nothing can be proven about what
a program will do before it's run. We need to be well-formed enough able to 
prove _some_ things, and then use other runtime metadata to enable caching.


## Compiled or Interpreted?

I don't know yet.

## Builtins

We include a prelude of builtin functions.

* TODO

## Doing the Right Thing

One problem with other interpreted languages is that doing the right thing in 
your scripts takes boilerplate.

Take using environment variables as an example. Let's say we need to use the
variable `TOKEN` in our build. It's a secret token, specific to a developer, and
there's no default we can provide. If `TOKEN` is unset, we need to stop the
build.

Here's one way to do that in Python.

```python
import os
tkn = os.getenv('TOKEN')
if tkn is None:
    raise ValueError("TOKEN must be set")
```

Ruby's `ENV` saves us the import, but not the check for `nil`.

```
tkn = ENV["TOKEN"]
unless tkn
  raise "TOKEN must be set"
end
```

This is one variable. What if your build needed several? Of course, we could do
the **wrong** thing and let our script fail later on, when we actually pass our
value to something. That's a recipe for confusion though.

Our language can provide a built in, variadic function for environment variables
that makes it easy to do the right thing.

```rust
let tkn = getenv("TOKEN")  // fail immediately if unset
let foo = getenv("FOO", "swag") // foo becomes "swag" if unset
```

## Example

```rust
// args is a builtin, magic list
let script_path = args[1]

fn shell_out(path) int {
    // shelling out yields an exit code, an int
    $(python path)
}

// call our function with the argument passed, ignore return value
shell_out(script_path)

```


