sbc - Use SSH Like a Boss!
==========================

Introduction
------------

``sbc`` stands for **Secure Back Channel** and is a little tool to help you using
SSH sessions. It looks like
`bcvi <http://sshmenu.sourceforge.net/articles/bcvi/>`_ but is more secure in
the command execution process and flexible when creating plugins. For now it
only works on Linux client machines (remote machines must run SSH daemon and
have Bash).

Sometimes when you are in a ``local-machine`` and connects via SSH to a
``remote-machine`` you need to copy some files between two machines, run some
time-consuming commands and other things. ``sbc`` helps you in these tasks so
you don't need to open other terminal windows/SSH sessions to do it or you can
be notified when some tasks are done, for example.

Are you lazy? **Watch the `sbc screencast <http://youtu.be/Rv55V_gfGEw>`_ and
learn how it works**.


Usage
-----

First, you need to `install sbc <https://github.com/turicas/sbc#installation>`_
in your machine and then install it in the remote machine - you don't need root
access in either machines, but need to run SSH daemon in both machines. After
installing in your machine, let's install in the remote one::

    user@local-machine:~$ sbc install other-user@remote-machine
    [sbc] Generating SSH RSA key without password (/home/user/.ssh/sbc_rsa)... [OK]
    [sbc] Copying private key and sbc executables to remote server... [OK]
    [sbc] Executing sbc-install... [OK]
    [sbc] Updating local authorized_keys (if needed)... [OK]

After installing it in remote, let's copy a file (``/tmp/some-file.txt``) from
the remote machine to the local machine, executing a command
**in the remote machine** - this command is just calling ``sbc`` with the
``cp`` plugin.

First, let's check if there is no file called ``/tmp/some-file.txt`` on local
machine::

    user@local-machine:~$ ls /tmp/some-file.txt
    ls: cannot access /tmp/some-file.txt: No such file or directory

Ok, now we'll connect to remote server using ``sbc ssh`` and list files in
remote home directory::

    user@local-machine:~$ sbc ssh other-user@remote-machine
    [sbc] Copying sbc metadata to server... [OK]
    [sbc] Creating back-channel and connecting... [OK]
    other-user@remote-machine:~$ ls
    some-file.txt

Now, copy the file to local machine's ``/tmp`` and disconnect::

    other-user@remote-machine:~$ sbc cp some-file.txt /tmp/
    some-file.txt                                 100%  545     0.5KB/s   00:01
    other-user@remote-machine:~$ exit
    [sbc] Cleaning metadata on server... [OK]
    Connection to remote-machine closed.

...and check if the file was copied to local machine::

    user@local-machine:~$ ls /tmp/some-file.txt
    /tmp/some-file.txt

When we execute ``sbc ssh`` it creates a back-channel so the remote machine can
connect to the local machine through SSH. This back-channel is used to call
``sbc`` plugins in local machine when you run ``sbc plugin_name`` in remote
machine. **All plugins are stored and run in your local machine** - they are in
``$HOME/.sbc/plugins`` and a plugin is just an executable file (yes, of course
you can use your preferred language to write one! ;-).

``cp`` is only one of other cool plugins shipped by default with ``sbc``. For
example, if you want to edit a file stored in remote machine using a editor
running in your local machine (so you can use your own configuration files,
for example), you can use the plugins ``vim``, ``gvim`` or ``gedit``. Let's
see an example with gvim::

    other-user@remote-machine:~$ sbc gvim some-file.txt

It'll run ``gvim`` **in your local machine** editing the file
``/home/other-user/some-file.txt`` that is stored **in the remote machine**
- and you'll execute the command to call ``gvim`` **in the remote machine**
(mind blowing? ;).

Now let's see another great plugin: ``notify`` - it shows in your **local
machine** (generally near the clock) a **notification sent by the remote
machine**. It is handy in cases when you need to run time-consuming commands on
remote machine and don't want to "watch" every minute if the command finished.
A command is worth a thousand of words::

    other-user@remote-machine:~$ tar -zcf /tmp/myhome.tar.gz ~/; sbc notify "hey, your targz was created!"


Installation
------------

Copy the executable script ``sbc`` to some directory in your ``$PATH`` and the
entire directory ``plugins`` to ``$HOME/.sbc/``. The following commands do
it for you (please **read** the commands before executing)::

    mkdir -p $HOME/bin $HOME/.sbc
    wget https://github.com/turicas/sbc/tarball/master -O /tmp/sbc.tar.gz
    cd /tmp
    tar xfz sbc.tar.gz
    mv /tmp/turicas-sbc-*/sbc $HOME/bin/
    mv /tmp/turicas-sbc-*/plugins $HOME/.sbc/
    rm -rf /tmp/sbc.tar.gz /tmp/turicas-sbc-*
    chmod +x $HOME/bin/sbc
    line_to_add='PATH=$PATH:$HOME/bin'
    if [ -z "$(grep $line_to_add $HOME/.profile)" ]; then
        echo $line_to_add >> $HOME/.profile
        source $HOME/.profile
    fi


Author
------

The core idea of this software was stolen from
`bcvi <http://sshmenu.sourceforge.net/articles/bcvi/>`_ and the rest
(flexibility + security) was idealized and coded by
`Álvaro Justen <http://blog.justen.eng.br/>`_ (with much help from
`Flávio Amieiro <http://flavioamieiro.com/>`_ and
`Kretcheu <http://www.kretcheu.com.br/>`_).


Copying
-------

``sbc`` is licensed under the terms of the `GNU General Public License version
3 <https://www.gnu.org/licenses/gpl-3.0.txt>`_ as published by the
`Free Software Foundation <http://www.fsf.org/>`_.
