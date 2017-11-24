# Working with Docs

The docs are written in reStructuredText (rST) and you can use [this
guide](https://thomas-cokelaer.info/tutorials/sphinx/rest_syntax.html)
as a quick primer. It is similar to Markdown and so, it should not be
hard to pick up.

## Prereqs
Use the build image. If you want to run things locally, you are on
your own but the build image configuration will tell you what packages
you need.

## Build Script
The following command will lint and build the docs
```bash
$K10/build.sh docker_run build_docs
```

## Developing Docs

### Visual Studio Code

If you are using Visual Studio Code, install the [reStructuredText 
plugin](https://github.com/vscode-restructuredtext/vscode-restructuredtext)
and use ```cmd+shift+r``` for a live preview.

### Other Editors

If you are using something like vim or emacs, you can make it easy to
get a live preview of your docs by running

```bash
sphinx-autobuild $K10/docs $K10/docs/_build/html/
```
and then pointing your browser to ```http://localhost:8000```

### Capturing screenshots

To get the browser dimensions right every time, Magnet was used to
slot my browser (Chrome) in the top-right corner of a 4K screen
(Dell P2715Q) running at maximum resolution and then centered for ease
of use. Cmd-+ was also used 3 times to zoom in. Yes, this is very
scientific.

### Spelling

If you use technical terms not found in the common US English
dictionary, please add them to the spelling_wordlist.txt file or to a

```
.. spelling::

  new-word
```

section at the bottom of your file to prevent the docs build failing.
