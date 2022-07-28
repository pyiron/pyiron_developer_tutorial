# How to setup a Dev environment

## The Live Cycle of a pyiron Patch

0. Something breaks, you get annoyed.
1. Check out pyiron locally (your laptop/workstation) and read the code to find out what breaks
2. Make a change and commit; either by
    - Do it locally and push the change to your checkout on the cluster, test it interactively
    - Do it remotely via remote mount (see below) or editor features (pycharm pro, vim, etc.)
4. Write unit tests in local checkout
5. Update documentation
6. Push to the github repository and open pull request (PR)
7. Have us pester you in code review, because we didn't like the thing you did on line 17
8. your PR is merged
9. Once a week new versions of the pyiron modules are released
10. After a day they end up on PyPI/conda
11. The pyiron/dev module environment on the cluster gets updated

## Connect to the cluster (for completeness) (Step 1)

(See also
[this](https://gitlab.mpcdf.mpg.de/eisenforschung/cm/documentation/-/blob/master/Cluster.md#Pyiron-Development-On-CMTI))

```bash
ssh -X -f -N -L 8000:127.0.0.1:30000 cmti > /dev/null && xdg-open
https://localhost:8000/user/zora/lab/tree/zora
```

1. `-X` enables X11 forwarding, allowing to start simple graphics programs ( no ovito :| )
2. `-f` makes ssh go to the background and not block your shell
3. `-N` do not execute a remote command, just forward ports
4. `-L` redirect local port 8000 (point your browser here) to remote port 30000 (this is where
   jupyterhub is listening)
5. `xdg-open` is the generic "open with whatever program" command on linux, use `open` on mac

How to do the port forwarding via the gateway machine (i.e. possible without VPN login)

```bash
ssh -X -J gatempcdf -f -N -L 8000:127.0.0.1:30000 cmti > /dev/null && xdg-open
https://localhost:8000/user/zora/lab/tree/zora
```

Difference is `-J`, which tells ssh to go via the `gatempcdf` host, before connecting to the actual
target `cmti`.

Put shell aliases for these!

These commands here do not specify the full hostnames to the cluster machines, because I have setup
host-sepcific configuration to my `~/.ssh/config`.
I recommond to do the same, put a block like
```
Host cmti
    User zora
    HostName cmti001.bc.rzg.mpg.de
    # not necessary but useful if you know you're not gonna be in MPIE networks, saves you the -J
    #ProxyJump gatempcdf

Host gatempcdf
    User zora
    HostName gatezero.mpcdf.mpg.de
```
(This would also allow you to easily set per host ids with `IdentityFile`, technical but Vincent
might like it if you do that)

If you don't want that replace the short hands by user@full.host.name in the rest of this document.

## Mount a remote file system with sshfs (Step 1)

Install [sshfs](https://github.com/libfuse/sshfs) (available on all linux distributions and mac,
windows works via WSL).

Works like a regular `mount` on linux, e.g.
```bash
sshfs cmti: ~/remotes/cmti
-o idmap=user,follow_symlinks,auto_cache,reconnect,ServerAliveInterval=20,Compression=no,ControlMaster=no
```

The first argument `cmti:` comes from `host:path` and specifies what remote system to mount.
The second argument is the local target directory, which should be empty.
This example here mounts my home directory on the cmti login node to the folder `~/remotes/cmti`.

1. `uid` technical option, makes file ownership of local and remote match up
2. `follow_symlinks` makes the mounted folder correctly respect symlinks. (symlinks are like
   shortcuts in windows but for linux, ignore if you don't know what this is)
3. `auto_cache` cache the filesystem (speeds things up)
4. `reconnect` make the mount survive network failure, as soon as you reconnect to the cluster the
   mount becomes available again
5. `Compression=no` make transfer faster when connected to a high speed connection (e.g. you mount a
   folder from your workstation to your laptop that are connected to the same wifi; do your own
   testing if it helps for connections to the cluster)
6. `ControlMaster=no` attempt to reuse an existing ssh connection to the target host if possible,
   necessary to make mounting work with connections that go via the gateway machine.
7. `ServerAliveInterval` makes it so that the server doesn't drop the connection when don't use the
   mount frequently

Put this in a shell alias!

To unmount this again run
```bash
fusermount -u ~/remotes/cmti
```
(or your graphical file manager)

Disadvantages:
1. bit fickle with prolonged network failures (i.e. mount in the office, then go home, reconnect
   from there)
2. when a connection is currently dropped, all access to the remote mount is frozen.  Might need to
   kill ssh if that happens.

For use with the gateway machine additional configuration is needed, put this in your `~/.ssh/config`

```
Host *
    ServerAliveInterval 120
    ControlMaster auto
    ControlPersist yes
    ControlPath ~/.ssh/socket-%r@%h:%p
```
(or append if a `Host *` block already exists)

You'll make your life much easier if you copy your public ssh keys onto the login node

```bash
ssh-keygen # if you have no key generated yet, but probably you have
ssh-copy-id host.name
```

Last word on ssh: check the manpage (`man ssh`, `man ssh_config`) when in doubt.  It is excellent.


## Side-Loading python modules on the cluster (Step 2)

Generally you'll work on the cluster with the `pyiron/dev` module loaded.
That loads all the pyiron dependencies for you but what if you want to test a change to
pyiron_contrib that you have made before it is merged and pushed to the cluster?

Python has a list of directories that it checks for modules whenever you import one.
You can check it with
```python
import sys
print(sys.path)
```
It is determined when the python process starts, but you can modify it during runtime yourself.

There's a few options how to modify this path persistently:
1. `export PYTHONPATH=...:$PYTHONPATH` in your `~/.profile`.  You'll need to restart your jupyter
   server for this to take effect.
2. Add a `*.pth` file to your `~/.local/lib/python3.*/site-packages`.  In it you can list folders
   that python should also search.  You'll need to restart the interpreter for this to take effect.  See
   [this](https://docs.python.org/3/library/site.html) for details.
3. When you have a python repository checked out with a `setup.py` you can simply do
   `pip install --no-deps -e .`.  This will put a link to the module in your python path, so that
   you can modify the module inplace and it is reflected whenever you restart the interpreter.  See
   [this](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-e) for details.
4. Last resort for when you don't have a `setup.py` is to create symlink from your development copy
   of the module directly into the site package directory above.  No need to restart the interpreter
   in this case.

The officially sanctioned way to do it is option no 1.
If you decide to work on a pyiron module you should therefore:
1. `git clone pyiron...` on your local machine
1. `git clone pyiron...` on the cluster
2. Do one of the above to add the module into your python path on the cluster.  Be aware that
   generally the actual module is inside the repostory in a folder with the same name.  That is, if
   you check out pyiron in my/folder/pyiron, you'll need to add my/folder/pyiron/pyiron to your
   python path.
3. Add the cluster repository as a remote to your local repository: `git remote add cmti user@cmti:path/to/your/pyiron`.
   Where the first argument is an arbitrary name of the remote and the second argument is the ssh
   path with which you access the remote

## How to write Unit tests (Step 4)

We use the builtin [unittest](https://docs.python.org/3/library/unittest.html) and
[doctest](https://docs.python.org/3/library/doctest.html) modules.
Their documentation is generally well written and you can find many examples online.

In pyiron modules our test code is within the `tests/` subfolder.
Run it with

```bash
python -m unittest discover tests/
```

from the repository root to run all tests (might take some time) or run specific tests with

```bash
python -m unittest tests/my/module/test_myfeature.py
```

Embedded doctests are run with

```bash
python -m doctest pyiron/my/source/file.py
```

though we also have a setup where the doctests are run from the unittest suite.

(I'm sure pycharm has a way of doing this for you.)

What makes a good unit test?

1. Test localized behavior of the specific module that you are testing (that's where the name "unit"
   comes from)
2. Think about the interface of the module that you are testing, what it promises and what it
   expects.  Add tests specifically to check these invariants.
3. Isolate what invariants you are testing into separate unit tests.  In a specific test assume that
   all the invariants that you are not testing hold.  This helps keep the tests short and
   self-contained.
4. Clearly document what invariants you are testing.  Use the `msg` argument of
   almost all `unittest` functions to transfer your intention to the reader.
5. Write unit tests that are independent of each other.
6. Do *not* test implementation details.

Tiny tip: When debugging why tests fail, running the tests locally and liberal use of the builtin
`breakpoint()` function is you friend.

(Show some `DataContainer` examples here.)


## Writing (Good) Documentation (Step 5)

We use google style docstrings.  See
[here](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html#example-google)
for what that means.

In a jupyter setup you can access docstrings of object with the `?`.  If that doesn't work or you're
not in a jupyter console use the builtin function `help()`.

You can build the documentation locally to check that your markup is correct.
First install the necessary dependencies in your local setup.

```bash
conda create -f .ci_support/environment-docs.yml -n my-pyiron-docs
pip install wcwidth decorator # work-around doc env is currently broken
```

(I like to put this in a separate environment, but feel free to put it in your normal pycharm env.)

Run

```bash
cd docs/
make html
```

The final html pages are then in `docs/_build`.
Point your browser at `docs/_build/index.html`.

Guide lines:
1. Document all arguments, return values and exceptions that a function may directly are indirectly
   raise (within reason)
2. In case of exceptions: document the circumstances under which they occur
3. Document the argument type and function return type with [type annotations](https://peps.python.org/pep-0484/).
4. Use proper sphinx syntax when cross referencing other functions, class or methods.  See
   [here](https://www.sphinx-doc.org/en/master/usage/restructuredtext/domains.html#python-roles) for
   details.
5. Document your intentions!  Why does a function exist, what is the rationale that it takes the
   arguments that it does and what can go wrong.  What promises are made (if any) if it does.
6. Add usage examples in the form of doctests! *Extremely* important for job classes and workflows.
   Nobody wants to read your code to figure out how to slot which thingie into what.


## Debugging & Profiling

The builtin [pdb](https://docs.python.org/3/library/pdb.html) should be your first stop for debugging.
Jupyter also comes with an improved version [ipdb](https://github.com/gotcha/ipdb).
In notebooks you can put `%debug` in front of a python line to start the debugger before the code starts running.

I've written about [profiling](https://github.com/orgs/pyiron/teams/pyiron/discussions/66) before.
