---
title: Packaging and distributing your Project
categories:
- General
layout: post
aside: true
---
Building a new module, library or a complete application is an exciting and, at the same time, long process. At some point in this development process, you need to figure out how to distribute this project. We will look at Python based projects, in some detail and also certain language agnostic options.

<!-- more -->

### What is covered?
Python, being a general purpose language, can be used in a lot of different ways - from a simple automation script to AI applications to robot control. And everyday the community figures out new ways to use this flexible language. Because of its flexibility, there exist many frameworks that have their own rules for deployment. In other cases, projects are built with more than one language and this brings in interesting challenges on how to package and deploy.

So, a single post cannot really cover the wide variety of mix-and-match that is possible with Python. But a whole website does exist, with a sole focus on providing various resources related to Python Software Packaging: [The Python Packaging Authority.](https://www.pypa.io/en/latest/) This group has also published [The Python Packaging User Guide](https://packaging.python.org/), which is, and I quote from the website, 
>_...the authoritative resource on how to package, publish, and install Python projects using current tools_

And it is a very useful resource. Definitely have a look at that. In this post, therefore, I will write from experience and look at some ways in which I have deployed Python projects.

### Distributing Pure Python Modules
A Pure Python module is nothing but a script, or a collection of scripts in a folder, that depend only on the Python Standard Library. The only requirement is that, the module should contain atleast, an empty `__init__.py` file at every importable level in the folder hierarchy.
Distributing them is as simple as zipping the folder and sharing it!

### Distributing a collection
A collection of Pure Python Modules again can be distributed simply by zipping them. But there is another recommended approach that handles modules with external library dependencies as well - The Python **`setuptools`** module. All projects that use this approach come with a `setup.py` file.

There are two main outputs that can be generated with this approach:
 1. Python Source Distribution (sdist)
 2. Python Binary Distribution (bdist)

The same `setup.py` file can be used to generate both.

#### Source distributions
A source distribution (sdist) is used when the project is a Pure Python Project. It is essentially a compressed archive of the source code of the project along with the build and installation related info. A user with this distribution can then run 
```
pip install /path/to/sdist
```
to install the packages.

[This page](https://docs.python.org/3/distutils/sourcedist.html) details out more information regarding the same.

#### Binary distributions
Even though sdist works fine for most cases, a better approach even in case of Pure Python packages is to use the binary distribution.
A binary distribution can handle non-Python code. It can carry the compiled artifacts in those cases. The instructions to create these can be specified using `setuptools` and `distutils`.

Python's official binary distribution packaging format is called Wheel. As it is compiled, a wheel is sometimes restricted to a particular OS version and to an extent, the language version(s). So, a single project can have multiple wheels for the various supported combinations.

Installing the wheel is also pretty direct:
```
pip install /path/to/.whl
```
By default, a project's wheel is what is preferred by the installer. In case the wheel fails, the installer falls back to the sdist and rebuilds the code. Therefore, while sharing it is always good to share both.

#### The setup.py file
A properly configured setup.py file can generate both sdist and bdist for a project. The most minimal setup.py file will look something like this:

```
import setuptools

setuptools.setup(
    name="your-package-name",
    version="version_number",
    author="Example Author",
    author_email="author@example.com",
    description="A small example package",
    long_description="A longer description, usually from a README,
    long_description_content_type="text/markdown",
    packages=setuptools.find_packages(),
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: License Type",
        "Operating System :: OS Version",
    ],
    install_requires=[
        'LibA>=1',
        'LibB>=2'
    ]
    python_requires='>=3.6',
)
```
But this is just barebones and will mostly work only for Pure Python Packages that can be distributed as is. In most other cases, a more detailed setup.py file is needed. Have a look at [Pillow's setup.py file](https://github.com/python-pillow/Pillow/blob/master/setup.py)
In some cases, like Django, [the setup.py file](https://github.com/django/django/blob/master/setup.py) will look different from above, at first glance.

A quick start guide on creating setup.py and other associated files can be found [here](https://packaging.python.org/tutorials/packaging-projects/)

Once you have everything ready, you could run
```
python setup.py sdist (for source distributions)
python setup.py bdist_wheel (for binary distributions)

python setup.py sdist bdist_wheel (for both)
```

These distributed can be privately distributed or made publicly available in the official Python Packaging Index: [PyPI](https://pypi.org/)

### Containerized distributions
Another way to distribute applications or projects is via containerization. With containerization, the whole filesystem that the project is running on, can be packaged and distributed. Thus, it is completely language agnostic. These tools are more popular on Linux distributions. But cross-platform support is available on a few.

There are quite a few options out there, the most popular one being Docker.
A layman's explanation of Docker :smile: : (source: [Reddit](https://www.reddit.com/r/ProgrammerHumor/comments/cw58z7/it_works_on_my_machine/))
{% include figure.html image="https://i.imgur.com/3eTKEZp.jpg" %}

The benefits are obvious:
 * All the dependencies are isolated
 * The installation steps need to be done only once 
 * The system is guarenteed to work even across OS versions and in some cases even cross platform

Now obviously, this might be an overkill for small python modules or projects. Containerization tools like Docker and Snaps help in larger systems. These are also nice for cloud deployments. Containers are used to deploy:
 * Web Services and frameworks
 * GUI Applications (Check out the Snaps of VNC and Postman)
 * Backend applications like databases ([Postgres Docker](https://hub.docker.com/_/postgres))
 * and many more!

There are repositories available for the various container types:
 * [Docker Hub](https://hub.docker.com)
 * [Snapcraft store](https://snapcraft.io/)
 * [Flatpak Hub](https://flathub.org/home)

Explore them to see the possibilities.

### To conclude...
As mentioned earlier, this post can talk about only so much. Containers work in most cases. But sometimes it may be too much or your application doesn't fit into any of these categories (Hardware programs for example). There are many other distribution options. 

If you are using a framework, it usually comes with certain guidelines on how to deploy the project. Sometimes, it is also possible that the deployment option has to be changed. It is always good to know, the various ways in which a particular kind of project can be packaged and distributed. 