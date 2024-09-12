# Namespace Test

I found that `poetry install` creates a virtual environment (or handles dependencies) not the same way as an environment managed by pip. IMO, poetry should be able to handle shared high level namespaces, the same way as pip does.

## Example issue

This repo includes three sub-folders, each representing a python library (a, b, and c). Each of these projects contain a top level package folder named **nstst** (namespace test). Within that, they include sub-packages named **spa**, **spb**, and **spc** respectively. 

The packaging is managed by poetry for each library and for convenience, they each already include a wheel file for testing.

Library b has a dependency to a. Library/project c depends on a and b.

When running `poetry install` in b, poetry creates a virtual environment and installs *lib-a*. It also creates some reference to the source code of *lib-b* so that when one activates a `poetry shell`, in theory the packages and modules from *lib-b* are "on the path". However, importing the spb sub-package fails. In contrast, when one installs *lib-b* via pip into a virtual environment, both sub-packages are available.

### Reproduce

To reproduce, run the following commands:

```shell
$ git clone https://github.com/FlowMatric/namespace_test.git
$ cd namespace_test/lib_b
$ poetry install
$ poetry shell
$ python
Python 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from nstst.spa import mod_a as a
>>> from nstst.spb import mod_b as b
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ModuleNotFoundError: No module named 'nstst.spb'
>>>
```

In contrast, run the following commands to compare it with a venv/pip managed environment:

```shell
$ cd namespace_test/lib_b
$ python -m venv .venv2
f@FK-22:~/git/namespace_test/lib_b$ ./.venv/bin/python -m pip install dist/lib_b-0.1.0-py3-none-any.whl 
Processing ./dist/lib_b-0.1.0-py3-none-any.whl
Processing /home/fkluiben/git/namespace_test/lib_a/dist/lib_a-0.1.0-py3-none-any.whl
Installing collected packages: lib-a, lib-b
Successfully installed lib-a-0.1.0 lib-b-0.1.0

$ .venv2/bin/python
Python 3.10.12 (main, Jul 29 2024, 16:56:48) [GCC 11.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from nstst.spa import mod_a as a
>>> from nstst.spb import mod_b as b
>>> a.hello_universe()
Hello, universe from library a!
>>> b.hello_universe()
Hello, universe from library b!

```

### Library C

As mentioned, this repo includes a third library (*lib-c*) which we can use to demonstrate the issue in a similar fashion as outlined above.
