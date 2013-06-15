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


Hooks
-----

These hooks are already written and might give you some hints how to implement
a new one.


cascade-push
''''''''''''


Synopsis

    ::

        git-hooks add cascade-push TARGET_BRANCH_REGEX REMOTE

Description

    This hook will push selected branch to a specific remote. A branch ref
    matching the TARGET_BRANCH_REGEX will trigger the command::

        git push REMOTE TARGET_BRANCH_REGEX

Options

    TARGET_BRANCH_REGEX

        Regex to pinpoint the which branch will trigger the hook

    REMOTE

        Remote alias to which the target branch will be pushed to.



release-on-tag
''''''''''''''

Synopsis

    ::

        git-hooks add release-on-tag TARGET_TAG_REGEX HOST DISTANT_REPO_DIR

Description

    On tag pushed matching the TARGET_TAG_REGEX, a ``tar.gz``
    export of the corresponding work tree will be sent via ``scp`` to
    HOST:DISTANT_REPO_DIR repository.

Options

    TARGET_TAG_REGEX

        Regex to filter tag which will trigger the hook.

    HOST

        Remote host that will be used for the ``scp`` command.

    DISTANT_REPO_DIR

        Remote directory (on the HOST) that will be used to specify destination
        location for the ``scp`` command.



bzr-push
''''''''

Synopsis

    ::

        git-hooks add bzr-push BZR_IDENT TARGET_BRANCH_REGEX BRANCH_NAME_SUBST_REGEX


Description

    On branch pushed matching the TARGET_BRANCH_REGEX, it'll be converted
    and pushed to launchpad on the account identified by BZR_IDENT.

    The branch name will be used and transformed thanks to BRANCH_NAME_SUBST_REGEX
    to create the target launchpad branch name.

    Note that you might want to test that you can actually push with the
    user account that will launch the hook. This might requires some
    setup to be made.

    This hook requires ``git-bzr-ng`` to be installed.

    This hook is to be considered early-alpha stage.


Options

    BZR_IDENT

        Launchpad account identifier (can be a team or a user account).

    TARGET_BRANCH_REGEX

        Regex to filter branch which will trigger the hook.

    BRANCH_NAME_SUBST_REGEX

        Substitution regex (ie:``s%^lp/(.+)$%\1%g``) that will be used to
        get the target bazar branch name from the triggering branch name.

        This is quite important as launchpad won't tolerate some characters
        as ``/``. So you should make sure to remove them.



