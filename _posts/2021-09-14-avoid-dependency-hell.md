---
layout:     post
title:      Avoid dependency hell in Python with venv and pip freeze
category: python
---

As a developer in the mid-2000s, dependency management was one of the more challenging aspects of my role at a national media company.

One example that comes to mind was the almost literal pain involved in deploying a DLL update to our sprawling .NET web application across a web farm[^1] of three Microsoft IIS servers.

[^1]: Wow, I haven't used the expression "web farm" for years.  Equivalent AWS-speak is more like "target group".

The update process was simple but achingly manual: take each server in turn, manually drag and drop the new DLL into the server's Global Assembly Cache (GAC) folder, and then add a reference to the version number of the DLL in the server's machine.config.

Even when diligently following these steps, a catastrophic "Server Error in '/' Application" would sometimes occur, bringing the whole web application crashing down followed by panicked calls from our editorial teams.

<figure>
  <img src="/assets/images/blog/server-error.png" alt="Server error">
  <figcaption>Server Error in '/' Application, once a .NET developer's blue screen of death.</figcaption>
</figure> 

If we were lucky, depending on how many servers had been patched at that point, the web application would stay up every other page refresh or so.  Not an ideal customer experience, even for circa 2005.

The underlying cause was invariably due to an errant reference to an older version of the DLL in a web.config squirreled away in some folder within the web application.  It was commonly referred to as DLL hell.

Fast forward twenty years and I was reminded of these past woes when trying to get some old Python code up and running on a new laptop.  Not for my actual job these days, just for fun.

On running the code, Python complained about umpteen missing packages from NumPy to Beautiful Soup.  Dependency hell.  Now while it's easy enough to install them manually using pip, you need to know the exact name and ideally the version.

```python
pip install beautifulsoup

ERROR: Could not find a version that satisfies the requirement beautifulsoup
ERROR: No matching distribution found for beautifulsoup
```

Huh?  Oh.. it's actually called beautifulsoup4. Got it.  Ahh, the latest version has broken my code.  Sigh.  Seem familiar?

Fortunately, there is a better way to handle this scenario, and it involves virtual environments, pip freeze and a file called requirements.txt.

Virtual environments.  It's good practice to create and activate a virtual environment in your working folder as soon as you start a new project.  

The venv module provides a way to do this.  Here's what its [docs](https://docs.python.org/3/library/venv.html) say:

> The venv module provides support for creating lightweight "virtual environments" with their own site directories, optionally isolated from system site directories. Each virtual environment has its own Python binary (which matches the version of the binary that was used to create this environment) and can have its own independent set of installed Python packages in its site directories.

It's simple to create a virtual environment:

```python
python -m venv venv
.\venv\Scripts\activate
```

...although the activate command is platform-specific so check the [docs](https://docs.python.org/3/library/venv.html) for the correct syntax.

Now, with a virtual environment in place for your project, the interpreter can see only the packages you've installed in that specific environment.  You can see exactly which ones by running pip freeze:

```python
pip freeze

beautifulsoup4==4.10.0
certifi==2021.5.30
charset-normalizer==2.0.5
idna==3.2
requests==2.26.0
soupsieve==2.2.1
urllib3==1.26.6
```

From there, simply feed the output from pip freeze into a file called requirements.txt.

```python
pip freeze > requirements.txt
```

This creates a definitive record of your project's required packages and version numbers - in the form of requirements.txt - which you and others can use to install those dependencies in future within a fresh environment.  Here's the install command:

```python
pip install -r requirements.txt
```

Had I followed these steps way back at the start of this particular Python project that was causing me grief, I could have used the requirements.txt to get up and running in no time.

Of course, the specific install command above can be useful in other places too.  If you happen to build Docker images, you can use it within your Dockerfile.

In summary... using virtual environments, pip freeze and requirements.txt provides a neat way to manage Python packages.  Their use certainly helps me avoid dependency hell, and I hope they help you too.
