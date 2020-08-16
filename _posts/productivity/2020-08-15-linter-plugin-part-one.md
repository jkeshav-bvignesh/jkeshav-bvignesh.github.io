---
title: Writing a Custom Linter Plugin - Part 1
categories:
- Code Quality
layout: post
aside: true
---
In the [last post](/linting), I had written about how a linter helps us write better code while also ensuring language / organization specific compliance rules. I also mentioned that we could write custom rules or plugins to extend existing linter capabilities. While the concepts will remain pretty much the same, the exact implementation of the plugin will depend on the language and the linter you are using. I will show you how we can write a new plugin for Pylint, a popular linter for Python.

#### The Requirements
First, you need Python installed. The Python version I am using is Python 3.6.9, but the version will not be an issue and the code should work across other Python 3 versions. Also, the OS I will be using for this example is Ubuntu 18.04.

Next, you need to install Pylint, if it is not already there. Open a terminal and run
```
pip3 install pylint
```

NOTE: Pylint can also be installed as a separate package by running

```
sudo apt install pylint
```
The major apparent difference will be that with the former approach pylint is associated with the python version and the latter installs pylint as a separate application. I suggest that you follow the former approach for now.
That's it! You can test out the installation by running the below command: 
```
python3 -m pylint <filename>
```
#### The Basics
Before we write a custom plugin, it's important to understand how Pylint works. How does Pylint understand the code structure? For most of it's checks pylint does this using a concept called Abstract Syntax Tree (AST). The plugins we write will be based on this.

An AST basically, scans your code and generates a tree where each node holds a small bit of the code. There is another tree in code analysis that does almost the same thing - the Parse tree. Both the trees are used in the lexical analysis phase of code compilation. I am not going to go into lexical analysis in detail as a complete understanding is not required to get started. The lexical analyser checks the semantic correctness of your code. It checks your syntax and whether it matches the rules of your language's grammar. This is where a parse tree and an abstract syntax tree differ. A parse tree will contain a lot more syntax information than an AST. In the lexical analysis process, an AST is derived from the Parse tree. But, we don't have to implement an Abstract Syntax tree from scratch to write a plugin. Most of the work is already done. We will understand more about the AST as we go through the implementation.

In Python, a standard library called **`ast`** provides many interfaces to work with code at this level. Pylint uses another library called **`astroid`** for its functionalities. We will use the same. Astroid provides more useful inferences about the various nodes and some nice interfaces that will make writing the plugin easier. Astroid is installed as part of Pylint. Using the interactive Python Shell let's see a bit about how astroid works. Let's start with importing the library

```
>>> import astroid
```

Now, let's parse an example code snippet. To do that we use **`astroid.parse`**
```
>>> parsed_output = astroid.parse("a = b + c")
```
This line by itself has no meaning. We don't know what a,b, c are, what data they contain and what the output is. If you type **`parsed_output`** on the next line, you will see something like this:
```
>>> parsed_output
<Module l.0 at 0x7f1ef38e2048>
```
What does this mean? In the previous line, we passed in a code snippet. As far as astroid is concerned, that is the whole code. In Python an isolated piece of code is a Module. This is the start of the tree. So, Astroid is telling us that we have provided it a module. So far, so good. Is it possible to see the whole tree? It is, using **`repr_tree()`**. Run:

```
>>> print(parsed_output.repr_tree())
Module(
   name='',
   doc=None,
   file='<?>',
   path=['<?>'],
   package=False,
   pure_python=True,
   future_imports=set(),
   body=[Assign(
         targets=[AssignName(name='a')],
         value=BinOp(
            op='+',
            left=Name(name='b'),
            right=Name(name='c')))])
```
Alright, A lot more information! Note the print. If you run the command without print, everything will be printed on the same line and it will be a bit hard to read. It says the module has the follow data associated with it. The names are self-explanatory. Our focus will be the **`body`** attribute. That tells you a lot more about that line we passed in.

It tells you that, it is an **`Assign`** node. The value of that node is the output of a **`BinOp`** (Binary Operation), where the operation is + and the left operand is a **`Name`** node called b and the right operand is another **`Name`** node called c!

This is AST parsing. This is how astroid is useful. Now as you can see, the AST has a collection of nodes and each node has some useful information about the code. The AST for the above code snippet will look like this:
```
            '='
            / \
          'a' '+'
              / \
            'b' 'c'
```

This idea of nodes and values is at the core of writing a new Pylint Plugin. There are two other useful features of astroid called **`extract_node()`** and **`infer()`**. I will show you what these are and how they can be used for writing plugins in the following posts.