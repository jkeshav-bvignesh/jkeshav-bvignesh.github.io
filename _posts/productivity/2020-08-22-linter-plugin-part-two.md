---
title: Writing a Custom Linter Plugin - Part 2
categories:
- Code Quality
layout: post
aside: true
---
Okay, let's write a simple linter plugin using the basics discussed [in the last post](/linter-plugin-part-one). This plugin will check for the use of eval functions in the code. Now Pylint already has this feature by default, but let's write our one of our own to see how easy it is to write one. Once we are done with this checker, Pylint should indicate all the places an eval function has been used. We are going to write an AST Checker to achieve our objective. As mentioned earlier, Pylint can also handle raw filestreams and tokens.

_NOTE: Pylint's [official documentation](http://pylint.pycqa.org/en/latest/how_tos/custom_checkers.html) walks you through most of the important concepts required to use Pylint and also write checkers._

#### The Checker
Let's start with a new class called TestChecker. The Pylint plugins need to be written in specific Checker classes that follow a certain format. This class will inherit from the BaseChecker class of Pylint. There is some boilerplate that needs to be filled up before we write the actual check.

``` Python
from pylint.checkers import BaseChecker
from pylint.interfaces import IAstroidChecker

class TestChecker(BaseChecker):
  __implements__ = IAstroidChecker
  name = 'test-checker'
  priority = -1
  msgs = {}
  options = {}
```
Right. Let's break it down. The most interesting ones are the **`BaseChecker`** and **`IAstroidChecker`**. The BaseChecker class, as the name implies, provides certain functionalities required by all checkers. This primarily includes the validation of the checker class that we implement. The IAstroidChecker is an interface for checkers that play around with the types of the various statements in the source code. As seen earlier, Astroid will give us the required information.

Next comes the **`name`**. This name functions as an ID for the checker. It will become more important when checker **`options`** are provided. For now, we will not go into that. The **`priority`** should be a number less than 0. Checkers are prioritized from most negative to most positive. This will become useful very soon.

**`msgs`** is the dictionary to which we add our error message. There is a particular format here too.
```
{
  'ERROR_CODE': (
    'BRIEF_DESCRIPTION_OF_ERROR'
    'CHECK_ID',
    'A MORE DETAILED EXPLANATION OF ERROR'
  )
}
```
If you have seen a Pylint Error report before, you will instantly recognize this. The ERROR_CODE, CHECK_ID and the BRIEF_DESCRIPTION will show up in the report in the below format:

```
path_to_file:line_no:col_no: ERROR_CODE: BRIEF_DESCRIPTION_OF_ERROR (CHECK_ID)
```
Let's add our custom linter message, like below:

```Python
EVAL_CHECK_ID = 'custom-eval'
    msgs = {
        'C0001': (
          'Avoid Evals at all costs!',
          EVAL_CHECK_ID,
          'A longer explanation on why eval should not be used.'
        )
    }
```

If you notice, I moved the **`custom-eval`** to a separate variable. This is for obvious reasons - to avoid changing it in multiple places. When our checker class grows we might find ourselves in a situation where we are reusing some of these message definitions and to might have to change the name to account for this. It's better to keep these isolated. The ERROR_CODE has a format. Stick with this for now. I will explain the format later.

Let's write the checker itself. This is the simplest part. The code is very simple.

```Python
def visit_call(self, node):
    if isinstance(node.func, astroid.node_classes.Name) and node.func.name == 'eval':
        self.add_message(
            self.EVAL_CHECK_ID, node=node
        )
```
What's happening here? The function name follows a standard. All functions that use the astroid interface to run checks must follow a simple naming style: **`vist_<node_type>`**. As eval is a function call, the corresponding astroid node type is **`Call`** and thus the name of the function. The complete list of all node types can be found [here.](http://pylint.pycqa.org/projects/astroid/en/latest/api/astroid.nodes.html)

This function has one parameter - node. Pylint handles most of the complexity for you. The code conversion to AST, parsing of the tree and calling the correct checker. At a high level, when Pylint encounters a node of type **`Call`** it will execute all visit_call functions of the various checker and that node is passed as an argument. These nodes have certain attributes that we will use to ascertain whether the node is indeed an eval node or not. The actual check is then, very simple. If it a eval node, show the custom message!

To understand the code that does this, we need to use the functions we saw in the last post. In a new Python Interactive terminal, run this:
```
>>> import astroid
>>> node = astroid.parse("eval(10)")
>>> print(node.repr_tree())
```
We can now get an idea of what an astroid node of type eval will have. The output should be something like this:
```
Module(
   name='',
   doc=None,
   file='<?>',
   path=['<?>'],
   package=False,
   pure_python=True,
   future_imports=set(),
   body=[Expr(value=Call(
            func=Name(name='eval'),
            args=[Const(value=10)],
            keywords=None))])
```

What is important to us is the body attribute. That tells us that this is a **`Call`** node. The name of the function is **`eval`** and the arguments to that function is **`10`** where **`eval`** is of node type **`Name`** and 10 is of node type **`Const`**. This is the information that is required to narrow down our call node to eval function and that is what is implemented.

```
If the call node.func is an astroid node class called Name and Name.name attribute is eval, then:
```
Is that instance check necessary? It will work without that in most cases, but it is safer to do that check.
Now we call the inherited **`add_message`** function and pass in the error ID and the node itself. Pylint handles the rest.

Alright! That wasn't difficult now, was it?

#### The Final Piece
So, our checker is done. How do we actually have pylint use this? It can be done is 3 simple steps

**Step 1:** add the below function in the same file as the class definition

```Python
def register(linter):
    linter.register_checker(TestChecker(linter))
```
This function is used to register this checker to Pylint.

**Step 2:** Add this file to your PYTHONPATH
```
export PYTHONPATH=/path/to/file/with/class/definition/:$PYTHONPATH
```
This will be valid only for this terminal session. A more permanent solution would be to add this export to your environment config file.

**Step 3:** Load it to pylint
When you run pylint, you can simply mention the name of the checker definition file and it will use it!

Assuming the class definition is stored in the file *test_checker.py*, you would run:
```
python3 -m pylint --load-plugins=test_checker filename
```

You can test out the checker by writing a simple test file with an eval command and see the Pylint output. I created a test.py file with one line: eval(10) and ran it through pylint. This is my output:

```
 python3 -m pylint --load-plugins=eval_checker test.py
************* Module test
test.py:1:0: C0114: Missing module docstring (missing-module-docstring)
test.py:1:0: W0123: Use of eval (eval-used)
test.py:1:0: C0001: Avoid Evals at all costs! (custom-eval)

----------------------------------------------------------------------
Your code has been rated at -20.00/10 (previous run: -20.00/10, +0.00)
```

SUCCESS! Our checker worked and it is indexed right below the Default eval checker. This is where priority comes in. Change that parameter value and see what happens to the result. Play around with the other parameters, change the error code and see where that leads you.

Writing checkers are very easy once you can correlate it with the corresponding astroid node types. There are more complicated checks that can be done. Play around and see what is possible. I will also probably show a more complicated example in the future.
In the next post, we will look at priorities, configuration files, error code conventions, IDE integration etc.