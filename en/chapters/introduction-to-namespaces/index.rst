==========================
Introduction to namespaces
==========================

--------------------
What is a namespace?
--------------------

Namespaces are used to give a limited view of the system to a particular
process. For instance, a `network` namespace can be used to give a process the
impression of only having certain network interfaces attached to the system. A
different example is the `mount` namespace which only exposes a selection of
the mounted file systems, or the `pid` namespace which gives a manipulated view
of the running processes on a the system. All of these namespaces have
different applications, which will be described in greater detail as this book
progresses.

In order to work with namespaces, we need to get some basics out of the way,
which is what we will start with in this introductory chapter.

Viewing the current namespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All processes start out in the same namespace, which is the same as saying that
starting out, there are no namespaces. The namespace a process currently
participates in can be seen by investigating the `/proc/<pid>/ns/` directory.
Below are some examples of this directory for a few different process IDs on my
system.


These are the namespaces of the init process:

.. code-block:: bash

    # ls -la /proc/1/ns/
    total 0
    dr-x--x--x 2 root root 0 Feb  5 21:47 .
    dr-xr-xr-x 9 root root 0 Jan 28 19:04 ..
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 ipc -> ipc:[4026531839]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 mnt -> mnt:[4026531840]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 net -> net:[4026531969]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 pid -> pid:[4026531836]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 user -> user:[4026531837]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 uts -> uts:[4026531838]

These are the namespaces of a vim process:

.. code-block:: bash

    # ls -la /proc/25288/ns/
    total 0
    dr-x--x--x 2 root root 0 Feb  5 21:47 .
    dr-xr-xr-x 9 root root 0 Feb  3 20:56 ..
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 ipc -> ipc:[4026531839]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 mnt -> mnt:[4026531840]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 net -> net:[4026531969]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 pid -> pid:[4026531836]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 user -> user:[4026531837]
    lrwxrwxrwx 1 root root 0 Feb  5 22:03 uts -> uts:[4026531838]

The `ipc`, `mnt`, `net`, .., files are special symlinks. The number in the
destination of the symlink is a representation of the inode it links to. This
inode can in turn be seen as an identifier of the namespace being referred to,
this means `ipc:[0]` is a different namespace than `ipc:[1]`. As already
mentioned, each namespace has its own view of the system, and two processes
belonging to the same namespace, as in the example above, will have a shared
view.

The question is then, how do we create a new namespace? Well, there is a system
call for that. The system call is called `unshare`, which may seem unintuitive,
but the name comes from the fact that namespaces are created when processes
exit their current namespace, thereby creating a new one.

Below is some example code which first calls the `unshare` system call on the
network namespace, and then proceeds to launch an executable within the new
namespace.


.. literalinclude:: code-samples/unshare.c
    :language: c
    :linenos:

If we use the above code to launch a bash shell, we can use the `ip` tool to
view the available network devices within our new network namespace.

.. code-block:: bash

    # ip link
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    2: eth0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT group default qlen 1000
        link/ether 50:7b:9d:39:84:79 brd ff:ff:ff:ff:ff:ff
    3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DORMANT group default qlen 1000
        link/ether 94:65:9c:90:ae:98 brd ff:ff:ff:ff:ff:ff
    4: tun0: <POINTOPOINT,MULTICAST,NOARP,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 100
        link/none
    # gcc unshare.c -o unshare
    # ./unshare /bin/bash
    # ip link
    1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

We can see that outside the namespace, four network devices are visible, but
once the current networking namespace has been unshared, only the loopback
device is visible. There are many interesting things to do with the networking
namespace, but we will leave those for a later chapter. it should however now
be clear how namespaces are used to limit the view of certain resources, and
how they are created.

Pinning namespaces
^^^^^^^^^^^^^^^^^^

.. todo:: Describe how bind-mounting a namespace causes it to survive the death
          of the process which created it.

Destroying and leaving namespaces
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. todo:: Describe how removing bind mounts destroys pinned namespaces, and how
          exiting a namespace suffices to destroy it if it has not been pinned.