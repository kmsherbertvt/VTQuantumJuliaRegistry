# VTQuantumJuliaRegistry
Hosts a `LocalRegistry` for Julia packages developed by the VT Quantum group.

## When to Use

The point of this registry is to enable a Julia environment (or package) to robustly use a package we've developed in-house.
For example, if you create an environment and you want to use ADAPT.jl, the most robust approach is to use this registry.

Note that Julia's package manager recently got an interface change that should eventually enable more sophisticated uses of unregistered packages.
Therefore, I anticipate this registry will not be needed in the future.
But right now, it is *essential* when you want to use an unregistered package that *itself* uses an unregistered package.
For example, ADAPT.jl uses PauliOperators.jl.

Strictly speaking, you *could* avoid using the registry even then, by manually adding each of the unregistered dependencies to your environment, even ones you won't use directly.
For example, trying to add ADAPT.jl will result in an error at first, but you could add PauliOperators.jl first and *then* ADAPT.jl.
That strategy should be considered a "hack"; using this registry is the preferred solution.

## How to Use


First, activate the environment from which you would like to use our VT Quantum packages.

(If you have no idea what that means, scroll down to the `Julia Environments 101` section.)

Next:

```
>] registry add "<this registry's github url>"
```

That's it.
Now you can add our VT Quantum packages by name. E.g.:

```
>] add ADAPT
```

Note that, if you weren't using this registry, you'd have to type out the github url instead (which may fail if the package you added has unregistered dependencies).

## How to Modify

Having prepared `LocalRegistry.jl` in your global environment, activate the environment of the package you'd like to register. Then:

```
> using LocalRegistry
> register(; registry="<this registry's github url>")
```

The `register` function clones the registry's github repo onto your computer, modifies the files to register your active environment, commits the change, and pushes it back onto github.
In order for this to work, you're going to need to have commit permissions on github.
It's probably best if we only register "official" versions of packages stored on the group github accounts, and only supervisors or very-trusted postdocs do the registration process.

Note also that the active environment *must* be associated with a git project (i.e. have a `.git` folder), and this git project *must* be tied to a remote url (i.e. have a github repo).
It is the remote url that gets registered as the "where to find this package" information.

Supposedly there is a way to register a project without having its environment active (e.g. directly registering a repo on github), but I couldn't get it to work.

## Relevant Documentation
- https://pkgdocs.julialang.org/v1/registries/
- https://github.com/GunnarFarneback/LocalRegistry.jl

## Julia Environments 101

Eh? What's an environment? What's a package? Here's a quick crash-course:

A "package" is the thing you get access to when you use the `import` or `using` keywords.
It's a bit of code someone's developed to provide types and functions to help you do something useful.
It's distinct from a "script", which is a bit of code someone's written to *actually* do something useful.

An "environment" is basically a list of packages that you have access to at the moment, i.e. that you can import.
From a fresh install of Julia, your environment contains only those those packages shipped out with the "base" language.
That includes many ubiquitous things like `LinearAlgebra` and `Random`,
    but it omits some other ubiquitous things like `FiniteDifferences`,
    and it omits other very useful things like `BenchmarkTools`,
    and of course it omits packages we've developed in-house like `ADAPT.jl`.

You can use the Julia package manager to add new packages to your environment.
The easiest way is to use `Pkg` mode in the Julia REPL.
First start a Julia REPL by entering `julia` into a command prompt, then tap the `]` character.
Your prompt should have changed from `julia>` to `pkg>`.
You'll often see online some instructions like
```
>] add BenchmarkTools
```
This is short-hand for "enter `Pkg` mode and then type `add BenchmarkTools`".
If you do so, Julia will download a bunch of information and then the `BenchmarkTools` package will be in your environment.

You can have more than one environment - `BenchmarkTools` will be added (and therefore available) in whichever environment is currently *activated*.
The active environment is indicated in `Pkg` mode with a label in parentheses, before the `pkg>`.
By default, the REPL will start with a *global* environment activated,
    and Julia keeps a separate global environment for every major version,
    so you might see something like `(@1.11) pkg>`.
Julia keeps the "list of packages" for global environments buried inside of your `.julia` install directory.

Alternatively, you can activate a different environment, like this:
```
>] activate ASpecificEnvironment
```
You should see the text in parentheses change accordingly - your prompt becomes `(ASpecificEnvironment) pkg>`.
This environment starts off completely empty, but once you add something,
    Julia will create a `ASpecificEnvironment` directory and populate it with a `Project.toml` file,
    which is the file that actually lists the packages.
It is generally advisable to have a different environment for every distinct project you work on,
    and you should have a low threshold for deciding that a new idea constitutes a new project.
The idea is to keep track of which packages, and which specific *versions* of those packages,
    you used to do your work.
It also helps a lot when trying to work from multiple computers,
    because installing all these packages on a new computer is as easy as pushing all your work (including the Project.toml file) to github,
    cloning it on your new computer, activating the environment, and running:
```
>] instantiate
```
Be careful, though - the REPL will let you use packages in the active environment ***OR*** the global environment,
    so it's possible your scripts were using something that had been added to your global environment, which didn't get instantiated.
Try to avoid that, by only adding to the global environment certain development utilities like `BenchmarkTools` or `LocalRegistry`.

Packages are themselves associated with their own environment,
    meaning every package comes with its own `Project.toml`.
This environment defines all the *dependencies* of the package,
    ie. all the other packages you'll need to install to get this one to work.
When you `] instantiate`, Julia will automatically fetch all these dependencies for you,
    so you don't need to worry about them if you are just *using* a package.
But if you are *developing* your own package,
    you will need to make sure that every single package you ever import (with `import` or `using`) gets added to the package environment.
Note that this *even* includes things that ship out with base Julia, like `LinearAlgebra` and `Random`.

The last thing to understand is that you can only `] add ...` a package by *name* if that name is *registered*.
That is, there's gotta be *something* which associates a specific package name (like `FiniteDifferences`) with the location of some actual code,
    usually [on Github](https://github.com/JuliaDiff/FiniteDifferences.jl).
By default, Julia uses the "general" registry, which is an official but low-barrier thing that we could totally add all our stuff to if we wanted.
But...that would sort of obligate us to maintain our code infrastructure in a holistic and long-term way,
    and I don't think we want to commit to that.
Instead, we can `] add ...` links to Github directly - this adds what is called an "unregistered dependency".
But as it turns out, Julia is not quite (presently) equipped to robustly handle nested unregistered dependencies,
    so the solution (provided by this repo) is to add our *own* registry, in addition to the general registry.
