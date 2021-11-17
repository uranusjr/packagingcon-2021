---
marp: true
title: 'Python Packaging: Why Don’t You Just?'
paginate: true
theme: gaia
class: lead
backgroundColor: #eee
style: |
    section > * { margin: auto; }  /* Horizontally center content. */
    section::after { font-size: 55%; } /* Smaller page number. */
    li + li { padding: 0.5rem 0 0 0; }  /* Space between top-level bullets. */
    li ul li { padding-top: 0; }  /* Don't space out nested bullets */
    pre code, code {  /* Override weird code styling. */
        background: none;
        color: var(--color-foreground);
        font-size: 90%;
        margin-left: 0;
        margin-right: 0;
        padding-left: 0;
        padding-right: 0;
    }

---

<style scoped>
h1 { line-height: 1.25; margin: 0 0 1rem 0; }
</style>

# Python Packaging:<br>Why Don’t You Just?

**Tzu-ping Chung** @ PackagingCon 2021

<!--
Hello, my name is Tzu-ping, and I'm going to talk about Python packaging.
-->

---

## Me

- Call me TP (uraunsjr.com)
- Taipei, Taiwan
- Apache Airflow @ [Astronomer](https://www.astronomer.io/)
- Python packaging (PyPA & others)

<!--
As usual, here's a bit about myself. My name is close to impossible to pronounce unless you're a native Chinese speaker, so please just call me TP.

I live in Taipei, Taiwan, and currently work for Astronomer to maintain and develop Apache Airflow.

I'm also a Python packaging contributor and a member of the Python Packaging Authority, or PyPA.
-->

---

## Python Packaging LOL

- So weird we must split it into topics
- How did this thing happen?
- What does the solution achieve?
- How do we improve the situation?

<!--
Python packaging! People love to hate it.

But I guess you know where this is going. Python packaging is far from perfect, but I don't think it's correct to say it's WRONG. So I'm going to pick some topics where some people think Python packaging is doing wrong:

* What was the use case Python was trying to solve?
* Why did Python decide to go with that solution?
* What disadvantages the solution brought?
* Can we overcome it without breaking the original use case?
-->

---

## Why O Why

- A ton of packages downloaded for no reason!
- PyPI should automatically build wheels!
- I can import my package but you can’t find it?

<!--
So here are the topics I'm going to cover.

[NEXT SLIDE NOW!]
-->

---

<style scoped>
h2 { opacity: 50%; }
li { font-weight: bold; }
li:not(:nth-child(1)) { font-weight: normal; opacity: 50%; }
</style>

## Why O Why

- A ton of packages downloaded for no reason!
- PyPI should automatically build wheels!
- I can import my package but you can’t find it?

<!--
When pip resolves package versions, it sometimes needs to download multiple versions of a package. People get annoyed when it ends up downloading five versions of tensorflow and wasting 2 GB of network data.

Well, the behaviour is not "right" or "good", but first we need to figure out why pip chooses to do that, in order to come up with a sensible solution.
-->

---

## Per-artifact Metadata — History

- [PyPI is just a bunch of tarballs](https://dustingram.com/talks/2018/10/23/inside-the-cheeseshop/)
    - Well, also zipballs
    - So simple it’s called the Simple API
- Conditionals in `setup.py`
- Wheel contains prebuilt metadata

<!--
The reason why pip needs to download the whole tensorflow package to get dependency data is because those data, and all other metadata, are only available in the package.

Python packaging dates way before package installer is a thing. At the time, a package should be self-contained so you can distribute it on your website. And this became Python's package interchange format.

To make things worse, the de-facto "standard" build script for Python packages at the time is a Python file, setup.py, if you want to know DEFINITELY what dependencies a package has, the only way is to actually run that build script. So pip doesn't really have a choice but to download the entire package, because it does not know what data the build script would need.
-->

---

## They may all have different metadata

```html
<a href="...">tensorflow-2.6.1-cp36-cp36m-macosx_10_14_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp36-cp36m-manylinux2010_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp36-cp36m-win_amd64.whl</a>
<a href="...">tensorflow-2.6.1-cp37-cp37m-macosx_10_14_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp37-cp37m-manylinux2010_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp37-cp37m-win_amd64.whl</a>
<a href="...">tensorflow-2.6.1-cp38-cp38-macosx_10_14_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp38-cp38-manylinux2010_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp38-cp38-win_amd64.whl</a>
<a href="...">tensorflow-2.6.1-cp39-cp39-macosx_10_14_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp39-cp39-manylinux2010_x86_64.whl</a>
<a href="...">tensorflow-2.6.1-cp39-cp39-win_amd64.whl</a>
```

<!--
This means that different binaries for one package, of one version, from one index, may require different dependencies.
-->

---

## Per-artifact Metadata — Advantages

- Every file server is a Python package index
- Just put artifacts in a directory!
- Install a package with a “direct URL”

<!--
But there are advatanges. Distributing Python packages is one of the simplest among ecosystems I've seen. Literally just a file server, or even a directory in a network drive is good enough. This makes collaboration extremely easy, it's extremely easy to push package usages inside an organisation.
-->

---

## Per-artifact Metadata — Improvements

- A way to say _I don’t have dynamic metadata_
- Serve metadata alongside with each artifact
- Detect and degrade gracefully

<!--
The solution is that, for binaries (or what Python people call wheels), metadata are static, so once pip downloads metadata, it knows exactly what to do. And we allow source archives (or what we can sdist) to optionally also use the same static format, so if pip downloads one of those, it does not need to run the build script.

We can also extend the Simple API, so if the server puts the package's metadata file under a special name. An installer can detect its existence, and avoid downloading the full package. This keeps the index simple.
-->

---

<style scoped>
h2 { opacity: 50%; }
li { font-weight: bold; }
li:not(:nth-child(2)) { font-weight: normal; opacity: 50%; }
</style>

## Why O Why

- A ton of packages downloaded for no reason!
- PyPI should automatically build wheels!
- I can import my package but you can’t find it?

<!--
We just talked about binaries a bit, and people complain about that as well. A lot of the Python users have come to expect being able to download binaries for everything, but sometimes those are just not available. For example, Python 3.10 was released like a month ago, but most of the projects out there still don't have wheels compatibile for it, so you need to build from source if you happened to just upgraded the base interpreter to 3.10. Why? OBVIOUSLY PyPI should just support it, right?

OK, so the main problem here is the cost. PyPI doesn't have (more) money because serving Python packages is expensive enough RIGHT NOW. But there are actually some technical issues as well...
-->

---

## [Infinite Wheelhouse Paradox](https://en.wikipedia.org/wiki/Hilbert%27s_paradox_of_the_Grand_Hotel)

- How many platforms does CPython support?
- Trick question! As long as it builds, it’s good
- There is a very long tail of platforms
- You can never build _all_ wheels, theoratically

<!--
Which I call Infinite Wheelhouse Paradox.

Python does not have the concept of support tiers, but whatever builds is "supported". So there's a very long tail of environments, some of them obscure, and some NOT OBSCURE but have interesting properties, like Android can't do multiprocessing properly, things like that. The possibilities are infinite.
-->

---

## Pull-based Platform Declaration

- Basically feature detection
- Declare needs for platforms to satisfy
- Platform PEPs are _Informational_
- As long as your binaries work for your users

<!--
This sounds awful, but is a good problem to have. Python being easy to build and run is one of the reasons it is so popular right now.

Instead of declaring what REAL-WORLD systems are supported, we create ABSTRACT DEFINITIONS that any system can CHOOSE to be. When a package is requested for installation, a tool can detect features like GLibC version, or whether pthread is available, to decide whether the environment fits a particular definition, and install a binary that SHOULD work.
-->

---

## Sufficiently Large Enough

- Make platform reflection easy
    - `sysconfig`
- Create a “common enough” template
    - `cibuildwheel`
- Simplify and externalise wheel-building

<!--
The modern development of pervasive CI availability also helps because those are essentially a definition what are "popular" platform people likely are looking for. So there's a tool called cibuildwheel you can set up on CI, and it will build your wheels for all the CI definitions you have.

This is what we do in engineering when we want infinite stuff, right? Infinity is impossible, but "sufficiently large" is practically good enough.
-->

---

<style scoped>
h2 { opacity: 50%; }
li { font-weight: bold; }
li:not(:nth-child(3)) { font-weight: normal; opacity: 50%; }
</style>

## Why O Why

- A ton of packages downloaded for no reason!
- PyPI should automatically build wheels!
- I can import my package but you can’t find it?

<!--
Next, how does pip know a package is installed or not? I copied a package to another place and pip says it's not installed or an older version is installed? The metadata storage system seems a bit wonky as well.
-->

---

## How did tools know what to uninstall?

- Trick question! They didn’t LOL
- Setuptools invented eggs (kind of like JAR)
- Turns out many packages don’t work inside a ZIP
- Let’s not ZIP them? Metadata become plain files.

<!--
We need to talk about how Python, the interpreter, manages packages. Or... how it doesn't.

Python packages are just a bunch of files and directories on disk. So for a long time, Python packaging tools literally CANNOT uninstall packages, unless you manually remove them yourself. And even THAT is difficult, because you can't even know what files were installed unless you read the source code, particularly setup.py.

So setuptools came up with an idea. Every package, is put inside a ZIP archive, called eggs, kind of like JARs on Java, that contains the source AND metadata, including a manifest of files. But a lot of packages out there aren't implemented well and have problems dealing with relative paths etc. in a ZIP, so eventually we got rid of the ZIP part, and just have flat files and directories.
-->

---

## Plain files are pretty nice

- Metadata as package resources (data files)
- Reflection is automatic via import search path
    - Disabling user-site, `PYTHONPATH`, etc.
- Are there any real technical downsides?

<!--
So yeah, metadata in Python are just files, and installing a package also installs a bunch of extra files that are logically a part of the package, although the package author never writes them.

But that's actually... good? Reading and writing plain files is a lot easier than maintaining a database interface, and this makes the metadata discovery logic exactly the same as Python's import logic, which is easier to both understand and maintain.
-->

---

# None of these makes sense!

<!--
OK, so now we know why those things happened, and they do kind of sound reasonable every step along the way now it's broken down like that. But it still does not really make sense they end up like this. Shouldn't we at some point, like, just shift course entirely? Take everything down and start all over? It's still mind-boggling why those little abnormities keep accomulating to this point.

AND YOU ARE RIGHT! It's still very weird.
-->

---

## Third System Effect

- First effort (`distutils`) went OK
- Second, not so much
    - You probably never heard of it
- More difficult to gather volunteers
- No central driving force

<!--
Again, going back in history. The first Python packaging implementation was a standard library module called distutils. It was kind of fine considering what was thought as best practice. But packaging paradigms change, so people start building things on it, like setuptools.

Gradually things get messy when you keep working on old old, so there was a drive to rewrite and consolidate things, called distutils2. But that didn't go as well, mainly because the scope was too big and it's difficult to get all the pieces in place in one shot while people are still using the old system. This is why Python packaging tools seem a bit of lagged behind, because the current tooling skipped an entire era.

This was when PyPA was created, and we want to avoid the mistakes. Instead of rewriting everything at once, things are done more iteratively. This feels slow since we need to more extensively discuss an addition to make sure it works within the existing boundaries, but is arguably faster in reality since we can roll out a usable thing when it's ready, instead of needing to implement more things from scratch and endure a long period before actually getting to the feature.
-->

---

## From `pip` to PEPs

- Set the ground rules for people to innovate
- Small, reusable libraries for standards
- Invite discussions to combine the parts

<!--
There are two parts to achieving this. First, we need to set up explicit rules, so new implementers don't need to copy pip bug-to-bug. Follow the standards, and if the standards and pip disagree, we fix pip. So we produced a bunch of PEPs describing how Python packaging currently works, and write a PEP for every new feature we want to add.

Second, we build libraries for common things. Traditionally, packaging tools are mostly monoliths, so new tools either need to rely on private API, or even just copy code, and we end up with a sort of Hynum's Law situation. This can be avoided if things are split into smaller pieces.
-->

---

## Python Packaging Platypus

- Evolutions make sense when they happen
- Special advantages for particular use cases
- Remember the lineage

<!--
So this is Python packaging! A while ago, I think at a PyCon I didn't attend, it's proposed that Python packaging should have a mascot, and a platypus seems like a perfect choice. It's weird to look, but everything it has makes sense and is needed, because evolution. It's super important to always remember the lineage, because things exist for a reason.
-->

---

<!-- _backgroundColor: #fff -->

<style scoped>
section { padding-right: 0 }
p { font-size: 70%; }
</style>

# Thank You!

https://monotreme.club

![bg right:60% 90%](https://monotreme.club/img/sticker.png)

<!-- And this is our platypus! That's all I have. Thank you! -->
