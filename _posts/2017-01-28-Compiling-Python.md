---
layout:  post
title: "Compiling Simple Python Extensions in 2017"
categories: cython, compiling python, python extensions, visual studio build tools
---
# Compiling Python extensions on Windows (in 2017)

Lately I've returned to my Monte Carlo tree search project, which is an implementation in Python of the Monte Carlo tree search method, applied to the game of Reversi (aka Othello).

Because the success of the Monte Carlo game agent is directly tied to how quickly it can churn through simulations, I decided to try compiling it with Cython.  It turns out doing so effectively doubles the speed of my agent.  I think this speed boost is primarily due to how I can compute using raw C integers instead of "Python Object integers".  Or maybe it is some Compiler Magic(TM), or both.

Compiling Cython-generated .c files on Windows is relatively straightforward nowadays.  There is a lot of information out there documenting how to do this with older versions of Visual Studio, or older versions of Python.  Here's what I have found to be the best way to do it with modern tools today, in 2017.  (I'm assuming these instructions are valid for any kind of Python C extensions, but I have only tested with Cython files.)

### Compiling Cython C (or C extensions) on Windows
I am using Python 3.6.0 64bit on Windows 10 64bit, with Cython 0.25.2 installed via `pip install cython`.

First, for good measure, let's compile our .pyx file into a .c file using Cython:

`cython -3 board.pyx -o board.c`

Now we have our C file to compile.

#### Download Visual C++ Build Tools 2015

We will get our compiler not from installing Visual Studio, but from the new [**Visual C++ Build Tools 2015**](http://landinghub.visualstudio.com/visual-cpp-build-tools).  Note that a preview for 2017 is available, but Python 3.6.0 was compiled against MSVC v.1900, which [corresponds to Visual Studio 2015](https://en.wikipedia.org/wiki/Visual_C%2B%2B#Common_MSVC_version) (2017 may also work, I do not know if the runtime changed).


This install should be about 4gb, compared to around 11gb if we had installed the full Visual Studio.  [If you already have Visual Studio installed, you should already have these tools installed.](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx)

Once the Build Tools are installed, navigate to the install location (for me, `C:\Program Files (x86)\Microsoft Visual C++ Build Tools`).  You should see a group of shortcuts that will launch different environments, each prepared to compile for a different target.  Open the one relevant to you ([you can read about the exact target specifications here](https://msdn.microsoft.com/en-us/library/f2ccy3wt.aspx#Anchor_0)).  Of course, most people in 2017 will want the x64 native target.

#### Compile
Once the environment comes up, change directory to the dir with your C extension file.

You will need to include the Python.h header during compilation, and you must also link in the Python lib file.  The syntax you want looks like this:

`cl /LD /I[directory with Python.h] [your C file] [the python36.lib file]`

As an example, here was the full command I ran to compile my file `board.c`:

`cl /LD /IC:\Users\Andy\AppData\Local\Programs\Python\Python36\include board.c C:\Users\Andy\AppData\Local\Programs\Python\Python36\libs\python36.lib`

If you receive any errors, check to make sure you are targeting the correct architecture -- you need to be using the exact same target as the version of Python you are running against.  If you are on a 64 bit machine but running a 32 bit version of Python, then you must target x86, etc.  Change targets by switching to the correct environment shortcut.

After this command completes, you should see some new files in your directory.  After compiling `board.c`, I see `board.exp`, `board.lib`, `board.obj`, and `board.dll`.  We can dispose of everything but the `.dll` file.  Since compiled Python extensions expect to have the file extension `.pyd`, we must change the name of `board.dll` to `board.pyd`.

#### Run it

Whenever I compile an extension module for Python, I hide/rename the existing `.py` file, so I know that when my program executes it is definitely calling the extension and not the vanilla Python file.  Hopefully, after following these steps, your program runs -- and hopefully it runs faster.

I don't have a comments section, email me with any questions.

#### Notes

These steps documented compiling a cython file 'by hand' with the Windows compiler cl.  If instead you are interested in setting up a Python package with a `setup.py` and distutils, [read here.](https://wiki.python.org/moin/WindowsCompilers#Compilers_Installation_and_configuration)

I referenced [this great writeup about evolving C/C++ runtimes](http://stevedower.id.au/blog/building-for-python-3-5/) by Steve Dower.  It is also very helpful with understanding the situation of linking Python extensions statically or dynamically with MSVCRT.