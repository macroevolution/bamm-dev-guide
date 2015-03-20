.. _documentation:

Update the documentation
========================

This chapter explains how to update BAMM's documentation,
including editing existing content, adding new content,
converting the documentation to HTML,
and finally uploading it online to `<http://bamm-project.org>`_.
The documentation is located in the *doc* directory of BAMM's root directory.
Most of the actual text of the documentation
is located within the *doc/source* directory.

BAMM's documentation was created using `Sphinx <http://sphinx-doc.org/>`_.
Sphinx is a tool that allows you to write documentation
in an easy-to-type format
(`reStructuredText <http://sphinx-doc.org/rest.html>`_),
and then convert it into HTML so it may be viewed as a website.
Therefore, you need to install Sphinx on your computer
before following the instructions in this chapter
(see `Installing Sphinx <http://sphinx-doc.org/install.html>`_).


Edit existing content
---------------------

#. Open the file you would like to edit.
   Source files are in *reStructuredText* format (extension *rst*),
   which are then converted to HTML format.
   Therefore, if you would like to edit the *mc3.html* page,
   you need to open the *mc3.rst* file.

#. Edit the file. Consult with the
   `reStructuredText Primer <http://sphinx-doc.org/rest.html>`_
   to learn how to edit *rst* files.
   We created a basic *rst* template file (called *template.rst*)
   in the *doc/source* directory to help you remember the most common commands.

#. If the BAMM version is changing, open *conf.py* and update *version* and *release*.
   This will update the text that appears in the top of the web browser on bamm-project.org. 


Add new content
---------------

#. To create a new page in the documentation,
   create a new file with the *rst* extension (e.g., *ou-model.rst*).

#. Edit the file following the *reStructuredText* format.

#. Open the file *doc/source/documentation.rst*,
   which contains the contents of BAMM's documentation.
   Add your new page to the list in the correct order
   (without the *rst* extension),
   which will appear in the table of contents as a new section.


.. _convert:

Convert to HTML
---------------

To convert the documentation to HTML,
type the following in the *doc* directory::

    make html

This will create a directory called *build*,
which will contain a directory called *html*,
which contains the website for the entire documentation.
To view the website, open the file *build/html/index.html* in a browser::

    open build/html/index.html

As you add or edit content to the documentation,
build the documentation often to check your work
before uploading it online.


Upload online
-------------

BAMM's documentation uses `GitHub Pages <https://pages.github.com>`_
to host its website.
The website is in a special branch called *gh-pages* in the BAMM project.
Anything in this branch is what the public sees
when they access BAMM'S website (`<http://bamm-project.org>`_).

#. Make sure that all edits in the documentation have been committed
   to the repository and that there is nothing left to commit
   in your current branch.

#. Convert the documentation to HTML (see :ref:`convert`).

#. Go to the root directory and checkout *gh-pages*::

       cd ..
       git checkout gh-pages

   If you view the contents of this branch,
   you will see that most are HTML files.

#. In case this local branch is old, update it with ``git pull``.

#. The newly built website is in *doc/build/html*,
   so we want to replace *almost* everything in the current directory
   with the contents of the new website.
   Delete everything except the file *CNAME* and the *doc* directory::

       rm -rf *.html _* *.inv *.js

   There may be other files or directories left over
   from another branch that were not committed. Delete these, too.
   Check which files these are with ``git status``
   and look at the *Untracked files*.

   If you accidentally delete the *CNAME* file, get it back with::

       git checkout CNAME

   If you accidentally delete the *doc* directory,
   you need to go back to the *master* branch and build it again.

#. Copy everything from *doc/build/html* to the current directory::

       cp -r doc/build/html/* .

#. Delete the *doc* directory::

       rm -rf doc

#. If you run ``git status`` you should see the files
   that will be updated on the next commit.
   Add and commit these changes to the repository::

       git add --all .
       git commit -m "Update documentation for version 2.2.0"

#. Push these changes::

       git push

   Within a few minutes, the online website
   should be updated with the new changes.
