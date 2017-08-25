# Creating documention with Sphinx:

0. Install sphinx: pip install sphinx sphinx-autobuild

1. Inside package (e.g vc3-client), create "docs" directory. 
Note: Make sure vc3-client was installed and not just cloned, hence you can use the modules and their dependencies (e.g vc3-infoservice) for a proper autodocumentation.
Otherwise, you will need to mock dependencies, which will be seen later (readthedocs section)

2. Inside "docs", type "sphinx-quickstart" with following settings (at least, what I used)
```
$ sphinx-quickstart 
Welcome to the Sphinx 1.6.3 quickstart utility.

Please enter values for the following settings (just press Enter to
accept a default value, if one is given in brackets).

Enter the root path for documentation.
> Root path for the documentation [.]: 

You have two options for placing the build directory for Sphinx output.
Either, you use a directory "_build" within the root path, or you separate
"source" and "build" directories within the root path.
> Separate source and build directories (y/n) [n]: n

Inside the root directory, two more directories will be created; "_templates"
for custom HTML templates and "_static" for custom stylesheets and other static
files. You can enter another prefix (such as ".") to replace the underscore.
> Name prefix for templates and static dir [_]: 

The project name will occur in several places in the built documentation.
> Project name: vc3-client
> Author name(s): http://www.virtualclusters.org/team

Sphinx has the notion of a "version" and a "release" for the
software. Each version can have multiple releases. For example, for
Python the version is something like 2.5 or 3.0, while the release is
something like 2.5.1 or 3.0a1.  If you don't need this dual structure,
just set both to the same value.
> Project version []: 1.0
> Project release [1.0]: september-demo

If the documents are to be written in a language other than English,
you can select a language here by its language code. Sphinx will then
translate text that it generates into that language.

For a list of supported codes, see
http://sphinx-doc.org/config.html#confval-language.
> Project language [en]: 

The file name suffix for source files. Commonly, this is either ".txt"
or ".rst".  Only files with this suffix are considered documents.
> Source file suffix [.rst]: 

One document is special in that it is considered the top node of the
"contents tree", that is, it is the root of the hierarchical structure
of the documents. Normally, this is "index", but if your "index"
document is a custom template, you can also set this to another filename.
> Name of your master document (without suffix) [index]: 

Sphinx can also add configuration for epub output:
> Do you want to use the epub builder (y/n) [n]: n

Please indicate if you want to use one of the following Sphinx extensions:
> autodoc: automatically insert docstrings from modules (y/n) [n]: y
> doctest: automatically test code snippets in doctest blocks (y/n) [n]: y
> intersphinx: link between Sphinx documentation of different projects (y/n) [n]: y
> todo: write "todo" entries that can be shown or hidden on build (y/n) [n]: y
> coverage: checks for documentation coverage (y/n) [n]: y
> imgmath: include math, rendered as PNG or SVG images (y/n) [n]: n
> mathjax: include math, rendered in the browser by MathJax (y/n) [n]: n
> ifconfig: conditional inclusion of content based on config values (y/n) [n]: n
> viewcode: include links to the source code of documented Python objects (y/n) [n]: y
> githubpages: create .nojekyll file to publish the document on GitHub pages (y/n) [n]: n

A Makefile and a Windows command file can be generated for you so that you
only have to run e.g. `make html' instead of invoking sphinx-build
directly.
> Create Makefile? (y/n) [y]: y
> Create Windows command file? (y/n) [y]: y

Creating file ./conf.py.
Creating file ./index.rst.
Creating file ./Makefile.
Creating file ./make.bat.

Finished: An initial directory structure has been created.

You should now populate your master file ./index.rst and create other documentation
source files. Use the Makefile to build the docs, like so:
   make builder
where "builder" is one of the supported builders, e.g. html, latex or linkcheck.
```

3. Edit conf.py, adding at the end the PATH to find desired modules to document
```
import sys
import os
sys.path.insert(0, os.path.abspath('.'))
sys.path.insert(0, os.path.abspath('../'))
```
This depends on where you will call sphinx-apidoc, so adding 2 levels, just in case.

4. Run: sphinx-apidoc, inside docs:
```
sphinx-apidoc -f -o . ..
```

5. Use the make file
```
make html
```
Note: You might see some warnings complaining about EOF structure in the files, etc. Not critical.

6. Check the vc3client awesome documentation:
```
vim _build/html/vc3client.html 
```

# Integrating it with Readthedocs

Since the above is only be seen by yourself, unless you have a web server to put your files to, you can integrate sphinx with readthedocs, the advantage of this is that:

- Documentation is regenerated with each git push 
- So, if a new class is added to an already existing module, this will be updated without any manual update.
- If a new module is created though, you either need to run sphinx-apidoc again, or update e.g vc3client.rst by yourself to include it.
- Free web server
- Most standard python packages use readthedocs.io

## Instructions:

1. Go to your "docs" directory and create a .gitignore file with the following:
```
_build
_static
_templates
```
readthedocs runs its own sphinx, so it only needs the sources.

2. Since vc3 packages are not in pip, you need to create a requirements.txt file with the dependencies.

For example, vc3-client/setup.py has the following:
```
install_requires=['requests', 'pyopenssl', 'cherrypy', 'pyyaml', 'vc3-infoservice']
```

Create requirements.txt  with the following:
```
requests
pyopenssl
cherrypy
pyyaml
mock
sphinx_rtd_theme==0.2.4
```
- Note that:
  - vc3-infoservice is not part of the requirements, that's because it won't find it in pip. 
  - We are requiring "mock", because we are going to need to mock the vc3-infoservice 
  - We request sphinx_rtd_theme 0.2.4, because there is a bug with earlier versions.

3. Change the "theme" in your conf.py. Look for html_theme in the script and replace/add the following:

```
import sphinx_rtd_theme
html_theme = "sphinx_rtd_theme"
html_theme_path = [sphinx_rtd_theme.get_html_theme_path()]
```
The default theme "alabasta" seems to have an issue with the vc3-client, so sphinx_rtd_theme is best.

4. Edit conf.py, adding the mock imports:
```
import mock
autodoc_mock_imports = ["vc3infoservice", "vc3infoservice.core", "vc3infoservice.infoclient"]
```

5. Edit index.rst and remove:
```
   :caption: Contents:
```
There is some bug related to that line in readthedocs, so most people remove it.

6. Upload to git

7. Go to: readthedocs.io and sign up

8. Import your project

9. Admin settings change the following:
Settings -> Programming Language -> Python -> Submit
Settings -> Documentation type -> Sphinx Html
Advanced Settings -> Requirements file -> docs/requirements.txt
Disable PDF, EPUB
