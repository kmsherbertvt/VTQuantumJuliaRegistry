# VTQuantumJuliaRegistry
Hosts a `LocalRegistry` for Julia packages developed by the VT Quantum group.

## Usage

If you want to use the most recent version of a Julia package developed in-house (like ADAPT.jl),
    the easiest strategy is to do the following in a Julia REPL:
```
> using Pkg
> pkg"registry add <this repo's github url>"
```
This code adds this registry to your personal Julia installation (i.e. it clones this repo into your `.julia/registries` directory).
All packages we've added to the registry can now be added to your own local Julia environments *by name* rather than by URL.

Once cloned, Julia automatically updates the registry as needed, so you need never look at it.
Note that this registry lives in your Julia installation, *not* your Julia environment.
That means you ***will*** have to do do this each time you want to run your work *on a different computer*,
    but you will ***not*** have to do anything new each time you start working *on a different project*.

## Adding packages to the registry

To register a new package so that others can add it by name,
    the easiest strategy is to **activate the package environment** and then do the following in a Julia REPL:
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


## When to Use

The point of this registry is to enable a Julia environment (or package) to robustly use a package we've developed in-house.
For example, if you create an environment and you want to use ADAPT.jl, the most robust and error-proof approach is to use this registry.

That said, it'll probably often feel easier to just add packages by url.
Indeed, you may well wish to do so, if you're trying to use a specific version of the code that we didn't explicitly register
    (perhaps because you're testing something minor in a specific commit that no-one else is likely to care about once you're done).
*Even in this case*, adding the registry above is likely to be helpful, if the package you try to add by url has its own unregistered dependencies.

Here is a hypothetical scenario, which should give a good idea of what this local registry is good for.

> I want to run a specific version of ADAPT, for a specific experiment.
> So, I set up a local Julia environment for just this experiment.
> Now I need to add the ADAPT.jl package to this environment.
> 
> **Without the registry:**
> I need to add the package by URL: `pkg"add https://github.com/kmsherbertvt/ADAPT.jl"`.
> Except, I'm trying to use a specific version, so I'd better include the commit hash: `pkg"add https://github.com/kmsherbertvt/ADAPT.jl#v8v981v"`.
> But now I get an error! ADAPT depends on PauliOperators, and Julia doesn't know where to find it!
> So now I have to add PauliOperators manually: `pkg"add https://github.com/nmayhallvt/PauliOperators.jl"`.
> Now I try to add the ADAPT url again.
> It works.
> 
> > NOTE: It is likely that an experiment involving ADAPT will require access to PauliOperators directly,
> >     so this doesn't seem all that big of a deal.
> > However, at time of writing, PauliOperators has its *own* dependency, on BlockDavidson,
> >     which *also* needs to be added manually.
> > And this is a big deal because BlockDavidson has *absolutely nothing* to do with ADAPT,
> >     to the point where *most* people running ADAPT code shouldn't even need to be aware that a thing called "BlockDavidson" exists.
> > But, if they want to run ADAPT, they have to manually add that package to their environment!]
>
> 
> **With the registry installed:**
> If I were just using the the latest version of ADAPT, I could simply `pkg"add ADAPT"`.
> But I'm after a specific version, which isn't registered.
> So I have to add by url after all: `pkg"add https://github.com/kmsherbertvt/ADAPT.jl#v8v981v"`.
> It works.

Note that Julia's package manager recently got an interface change that should by all rights enable adding by url to just work.
But it doesn't yet, and I'm tired of waiting!
Nevertheless, I have hope this registry will not be needed in the future.

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
