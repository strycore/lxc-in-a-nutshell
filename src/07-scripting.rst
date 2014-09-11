*****************************
Scripting with the Python API
*****************************


Using the LXC API
=================

Besides a standard LXC setup, you'll need to install the package
python3-lxc. As the name suggests this library uses Python 3.4. If the
python3-lxc package isn't available on your distribution, you can find the
pyhton package in the official lxc source tree.

Do not install or use libraries you could find on PyPI or Github. Most
haven't been updated in a very long time and have been obsoleted by the
official API. (specifically the lxc4u and pylxc projects)

Once the LXC python library is installed, you can test it works in the
(i)python3 shell::
    In [1]: import lxc

    In [2]: lxc.list_containers()
    Out[2]: 
    ('myproject',
     'database',
     'email')

If you don't see any containers and you know there are present, it's
because you are running python with a regular user. Unless you are using
only unprivileged continers, the python api requires root access just as
the shell commands do. you should be running::
    sudo ipython3
or::
    sudo my-script-using-lxc.py
