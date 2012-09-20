=========
git-hooks
=========

``Git-hooks`` provides a simple shell framework to manage your git hooks. 

This project is quite new and only supports the ``post-receive`` git hook,
implementing the other hooks could be quickly implemented.


Sample usage session
--------------------

::

  git-hooks add release-on-tag [0-9]+.[0-9]+.[0-9]+(-.*)? /var/www/packages

Will create a file ``/var/www/packages/mygitrepos-0.0.1.tar.gz`` when you'll push
a tag ``0.0.1`` on this repository.

::

  git-hooks add sync-workdir stable www.example.com /var/www/myrepos

Will maintain git repository ``www.example.com:/var/www/myrepos`` up to date
with ``stable`` branch of current repository whenever you'll push change on
this branch.

Both of these tags could live on the same repository without problems.


Usage
-----

Before using system wide ``git-hooks``, please set up the general git variable::

  git config --system git-hooks.install-dir MY_GIT_HOOKS_BASE_DIR

Once this is done, you are free to run the given previous examples.

Help is provided at each levels::

  $ git-hooks
  You should provide a sub-command.

  This is the list of supported sub-commands:
      git-hooks add HOOK [HOOK-OPTIONS...]
      git-hooks init

  $ git-hooks add
  You should provide a hook name.

  These are the available hooks:
    - release-on-tag
    - sync-workdir
    - update-publish-dir

  usage: git-hooks add HOOK [HOOK-OPTIONS...]


Internals
---------

Files and directory
'''''''''''''''''''

In an existing git repository::

  git-hooks init

will setup the necessary files to manage hooks with ``git-hooks``. Please remember
that this command is NOT required for any examples to work, as it is executed
automatically if not already done by any previous ``git-hooks`` command.

Init will create a generic hook file in the ``.git/hooks/<git-hook-name>``
files that sets up the framework, and a directory for each hook.

For the moment only the ``post-receive`` hook has been implemented. So it
creates a ``.git/hooks/post-receive`` hook file, which will parse the
``~/.git/hooks/post-receive.d/`` directory for individual hook, each one in a
file.

Adding a hook
'''''''''''''

Hooks are standalone shell script ending with ".hook" providing some predefined
shell function. Hooks are found in the base installation of ``git-hooks``, in the
``hooks`` subdirectory. This file provides the control part of the hook.

When calling ``git-hooks add myhook`` you'll trigger the ``install`` function of your
hook, and it's job is to create a "myhook.conf" file. This file configures the behavior
of your hook.

``post-receive_ref`` function will be called upon each reference received by git.

``post-receive_post`` function will be called once all ref has been received.
