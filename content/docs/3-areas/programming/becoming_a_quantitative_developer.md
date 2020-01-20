---
title: "Becoming a Quantitative Developer (Deprecated)"
date: 2019-04-25
---

# Table of Contents
1. [Introduction](#introduction)
2. [Getting Started](#gettingStarted) 
3. [Resources](#resources)
    1. [Programming Languages](#programmingLanguages)
    2. [Editor](#editor)
    3. [Version Control](#versionControl)
    4. [Software Development](#softwareDevelopment)


# Introduction <a name="introduction"></a>
This page serves as a learning guide for the important skills of a quantitative developer. It is primarily inspired by the QuantStart [Self-Study Plan for Becoming a Quantitative Developer](https://www.quantstart.com/articles/Self-Study-Plan-for-Becoming-a-Quantitative-Developer). The skills are divided in four rough and overlapping categories:
1. **Programming**: The ability to solve a problem in a specific programming language.
2. **Editor / IDE**: The ability to get the most out of your editor or IDE, which makes a large difference to productivity.
3. **Version Control**: Good version control practices. No more "Save As".
4. **Software Development**: Knowledge of how to engineer good, maintainable software as well as how to manage software-based projects.

The intention is to build a solid foundation in each of these areas and then specialize as needed. Our goals are three-fold:
1. **Become productive as quickly as possible.** This is to justify the investment both to ourselves and our employers as necessary.
2. **To learn together.** The learning process works best with other people. We are all going to use the same tools and commit to the same deadlines so that we can discuss the problems that arise and help each other out.
3. **Develop skills that are relevant to our further career development.** As much as possible, we want to learn skills that are *future proof* and not only specific to our current role and environment.

In light of these goals, I cautiously make the following recommendations: For programming languages, we will learn [Python](https://www.python.org/) and C++. As an editor, we will use [Visual Studio Code](https://code.visualstudio.com/). In terms of version control, we will learn the ubiquitous [Git](https://git-scm.com/) and host our personal learning projects on [GitHub](https://github.com/). I don't know much about software development at this stage, so we'll stick to the recommendations made in the self-study guide above.

# Getting Started <a name="gettingStarted"></a>
This guide assumes that you're using Windows. You'll need to install the following software:
* [Visual Studio Community Edition](https://visualstudio.microsoft.com/downloads/)  (For C++ development only, and you'll need admin rights)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Git for Windows](https://git-scm.com/download/win)
    - The default options should all be fine.
* [Miniconda for Python 3.x](https://docs.conda.io/en/latest/miniconda.html) (We'll use this to manage our Python environments)

Once all the necessary software is installed, we'll need to set up some things.
The first thing to do is go create a personal [GitHub](https://github.com/) account so that you can start practicing your version control as we work through the exercises. Then we can start setting up our development environments.

## Getting Started with Visual Studio Code and Python
To get started, we will work through the official [Getting Started with Python in VS Code](https://code.visualstudio.com/docs/python/python-tutorial) tutorial, but first note the following:

* VS Code extensions can be installed by clicking on the `Extensions` button on the left-hand side of the VS Code window and then searching for the appropriate extension and clicking `install`.
* I recommend you create a single folder for all your python code, with your projects as subfolders. For example, `C:\dev\python\hello` for the ''hello'' project in the tutorial.
* When the official tutorial says ''At the command prompt or terminal'', we will use our Anaconda prompt, which was installed along with Miniconda. I recommend you pin the Anaconda prompt to your start menu, or add it as a desktop icon.
* When the tutorial discusses selecting the Python interpreter, you should select the one containing `('base':conda)`. This is because we are using Conda as our Python package and environment manager.
* On your first try, consider skipping the section on [debugging](https://code.visualstudio.com/docs/python/python-tutorial#_configure-and-run-the-debugger), and rather move on to the example below. Although configuring the debugger is very useful, and we will use it later, it can be intimidating. 
* DON'T follow the instructions in the [Install and use packages](https://code.visualstudio.com/docs/python/python-tutorial#_install-and-use-packages) section of the tutorial! We won't be mixing virtual environments with our Conda environments. See below instead.

### Packages and Environments with Conda
Following on from the tutorial, create a new file called `standardplot.py` and copy and paste the following code:

```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 20, 100)  # Create a list of evenly-spaced numbers over the range
plt.plot(x, np.sin(x))       # Plot the sine of each x point
plt.show()                   # Display the plot
```

This code won't run, because the `matplotlib` _package_ is not installed in the current _environment_, known as `base`. So we're going to create a new environment for our program, activate it (switch context to it) and then install `matplotlib`. Then we can select the new environment as our Python interpreter and run the above code.

In the Anaconda prompt (the one you used to start VS Code, NOT the integrated terminal inside VS Code), create a new environment called `tutorial` using the Conda command:

```bash   
conda create --name tutorial
```

(If you get a `bad handshake` error, check out [this link](https://github.com/conda/conda/issues/4930).)
Then switch to the environment, by activating it:

```bash   
activate tutorial
```

Now we'll use Conda as a package manager to install the `matplotlib` package in our new `tutorial` environment (along with all its dependencies):

```bash
conda install matplotlib
```

Once this is completed, restart VS Code. Use the command palette to select a Python interpreter (as we did in the start of the tutorial), but now select the interpreter associated with your new environment, `tutorial`. You'll now be able to run the new file and use the `matplotlib` package.

For a handy reference to the Conda commands, see the official [cheat sheet](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf). To learn more about using Conda as the Python environment and package manager in VS Code, see [Using Python environments in VS Code](https://code.visualstudio.com/docs/python/environments#_global-virtual-and-conda-environments).


# Resources <a name="resources"></a>
The idea is to use reference material that comes recommended by experienced developers (thanks Reddit and StackOverflow) and is established enough that a consensus has been reached on the quality. We want to be idiomatic from the start: let's learn the language _as if_ we didn't know any other.

## Programming Languages <a name="programmingLanguages"></a>
### Learning C++

* "Programming: Principles and Practice Using C++" by Stroustrup.
* "Accelerated C++:  Practical Programming by Example" by Koenig and Moo.
* "Effective C++" by Meyers.

### Learning Python

* [The Official Python Tutorial](https://docs.python.org/3/tutorial/index.html). 
* "Effective Python: 59 Specific Ways to Write Better Python" by Slatkin.
* "Python Cookbook: Recipes for Mastering Python 3" by Beazly and Jones.
* "Fluent Python: Clear, Concise and Effective Programming" by Ramahlo.

## Editor <a name="editor"></a>
I have not yet found a project- or outcome-based VS Code tutorial and this is ideally what we would've like to do first. Perhaps we should make one.
* [VS Code Can Do That?!](https://vscodecandothat.com/)

## Version Control <a name="versionControl"></a>
* [Learn Git in the Browser](https://learngitbranching.js.org/) by Github.
* [Pro Git](https://git-scm.com/book/en/v2) by Chacon and Straub.

## Software Development <a name="softwareDevelopment"></a>
* "Code Complete: A Practical Handbook of Software Construction" by McConnel.
* "Clean Code: A Handbook of Agile Software Craftsmanship" by Martin.
