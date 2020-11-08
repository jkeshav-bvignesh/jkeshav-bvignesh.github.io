---
title: Could you Version Control Your Entire Computer?
categories:
- General
layout: post
aside: true
---
I recently had to reset my laptop. During this reset, I decided to find out if it possible to version control a lot more than just code.

<!-- more -->

* Table of contents.
{:toc}
We have all been here at some point - The system has some serious problem and the easiest solution is to _wipe everything, do a clean install, and restore the backup!_ Pretty easy, if you had a good backup policy in place.

[_The Pragmatic Programmer_](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/), in it's chapter on Version Control, hinted at this very situation and how Version Control can help. So, I did my usual backup routine and proceed with a clean install. Once everything was set, I looked at what else could be version controlled when I started with a "clean slate".

> If you haven't already read the Pragmatic Programmer, you should! It will be worth your time!

## What was backed up and where
The most obvious component - the code - is already in a remote repository. Along with the code, the virtual environment configuration was also Version Controlled. But that was it. Nothing else, in the system was under version control.

The user data to an extend was backed up in a cloud storage. This included the majority of the documents and some software builds. I had also created a last minute list of all installed software and extensions in a text file and saved it.

But most of the system configuration was lost. Thankfully, the important ones were noted down in one form or the other. I could have gone through each system setup and software configuration and made a copy, but a lot of metadata would be lost.

## How Version Control fits in
Let's look at the various components and see how we can automate the backup in a Version Control System. I mostly use a Linux distribution, both at home and office. But the concept should be applicable for all OSes
### Installed Apps
The simplest way is to maintain a list of all installed apps in a text file. The idea is to automatically install all the software in this list, when required. Should be simple, right?

Turns out, it was slightly more tricky. I had applications installed using three different mechanisms:
* via APT
* via Snap
* via Flatpak

#### Snap Apps
This one was the easiest. The above logic works directly. I could do a `snap list` and store the app names. Later on, when required a simple shell script that loops over the list and installs the applications worked. But maybe a more elegant solution is needed to accomodate the variety of install options.

#### Flatpak Apps
I had only one app that was packaged in this format. A shell script would again work here. But I also found an alternative using the other package format, so I didn't require it anymore.

#### Apps installed via APT
This is where things got a little bit tricky. Most of the default apps, could be installed simply by running `apt install package-name`. But in many cases, a specific repository had to be added, a key had to be used or it was install directly from a package file. Therefore simply maintaining a list of installed apps wouldn't work.

#### Shell Scripts - A better solution
Finally, I ended up writing a shell script that details the installation procedure for all my apps. The common thread among all this was that the complete installation could be done directly from the command line. This shell script can now be version controlled. The workflow, I follow now is this:
* Install an app using any one of the above methods
* Add the commandline steps in this shell script
* Commit it!

I see a few major advantages, with this approach:
* I don't have to worry about the packaging format.
    - It works even for apps that are distributed as compressed archives
    - Unzip and add an alias. All automatic!
* My entire installation procedure is automated!
* I can now have a history of all installed apps by date in Version Control
    - Rollbacks are simple

For now, I will be sticking with this approach. Is there a better approach? Let me know!

### Docker Images
This one had a pretty direct solution as well. **Version control the Dockerfiles!** I already had most of my docker images described in a Dockerfile, so this was not a problem.
But, for the sake of completion, the advantages of backing up the Dockerfile over the image itself are:
* Much less space
* It's a text file. So version control is easy
* Easier to share
* The docker images built will always be of optimal size
    - There are times a development docker ends up getting shared. This creates unnecessarily large images.
* Most of the time, the `Dockerfile` is part of the code and can be Version Controlled to accommodate changes
    - A particular version of code, may only require a particular subset of dependencies
    - You ensure optimal builds

### Documents
This may sound a bit extreme, but once you realise it's usefulness, I hope you will see the beauty of it all.
Most of the time, the documents we create depend on some word processor like Microsoft Office Apps or LibreOffice Apps. This creates a dependency that is actually not needed. Also, the final file that is created (e.g.: .docx, .odt) are binaries that can't easily be version controlled.

What is the solution? **Use Markdown!** Okay, before you react, all the **_content in this blog is in Markdown and completely Version Controlled!_** Think about that.

Markdown offers a variety of format indicators, that can then be read by some template engine and formatted however required. In this case, for example, HTML templates are populated with this content by Jekyll automatically.
The same can be done for other formats. You can convert Markdown to PDF, DOCX, LateX, HTML etc. There are softwares, plugins, extensions, scripts that do that for you.

You still have a dependency on Markdown. But this is a more flexible and lightweight dependency. And the representation is pure text. And as you must have realised by now, **text is good for Version Control!**

Now there maybe situations wherein, you can't find a good template conversion software or a conversion to the extension is not possible, in those cases you still have to maintain a copy of the file in that format. In those cases, you can still have backups in your cloud storage. These days, I have noticed that, these services provide some sort of Version Control options too. You can retrieve an older version of the document based on when you uploaded it. But it doesn't give you all the benefits of a proper Version Controlled System.

### System Configuration
Code, Apps, Docker Images and Documents. Well that's most of the useful data. This leaves us with the configurations. There are two kinds of configurations here:
* App Configurations
* System Configurations

#### App Configurations
This is completely app specific. Most apps stored the User configurations in some form of text file. There will be an option to export these configurations. You have to figure it out based on the apps you use. VSCode configurations, for example, can be stored as JSON files. The various extensions that are installed can be listed and reinstalled via commandline.

```
code --list-extensions # lists all your extension ids
code --install-extension <extension id> # will install the extension.
```
A simple shell command to loop over a list in a text file is all that is required to bring back VS Code.

#### System Configurations
This part is going to specific to Linux. I don't know how this part works in Mac or Windows. As with everything else in Linux, configs are also text files. Almost all your system configs are stored in `/etc`. There are many approaches to backup these configurations. The most direct way is to save all the config files (usually with a `.conf` extension) in version control.

But the catch is you loose certain metadata. One important aspect of these configuration files are the permissions. A simple copy-backup will only save the content but loose this critical information. This is a well known problem, and there are solutions such as **etckeeper** that can help. [Etckeeper](https://etckeeper.branchable.com/) creates a git repository inside `\etc` and maintains all this additional info.

Some of the things I described earlier, such as app installs and configs, are actually stored in this directory. Version Controlling this directory would solve the problem to an extent. But there are some complexities to those installs, that I felt that a shell script was still required.

### Is it really worth it?
It is a lot more work, no doubt about that. It can also feel a bit extreme, for a personal system. But the most important point is that, like everything with version control, it is a **_one time setup._** The benefits of setting up something like this is many-fold. You are in-effect creating snapshots of your system, but, these snapshots are lightweight, more flexible and easily maintainable.

And most importantly, once we start doing it on our personal system and experience the benefits, it will become a habit. Pretty soon, we will see how this can be applied at other places, like at work. Think about a version controlled server with custom configurations. Would it be helpful? I think so.

Of course, not everything can be version controlled and they need not. Multimedia content is a good example. But these don't change. There is only one state and a cloud backup is more than enough. A combination of Cloud storage and Version Control as a backup measure is, in my opinion, definitely worth it!