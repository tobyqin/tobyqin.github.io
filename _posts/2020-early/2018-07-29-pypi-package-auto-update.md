---
title: Auto Update Your Pypi Package
categories: [Tech]
tags: [python,pypi,pip]
date: 2018-07-29
---

Sometimes we mgiht want to make our package update to latest version, let me show you how do I accomplish this.

<!-- more -->

## Determine Versions

We have to determine current installed package version.

```python
def get_pkg_version(name):
    """Get current version of a installed pip package."""
    import pkg_resources

    try:
        return pkg_resources.require(name)[0].version
    except pkg_resources.DistributionNotFound:
        return None
```

Then detemine latest published version, if this is an internal package, you have to implement a custom method like bellow.

```python
def get_latest_version(name, server='https://pypi.org'):
    import re
    import urllib.request, urllib.error

    try:
        content = str(urllib.request.urlopen('{}/simple/{}/'.format(server, name)).read())
        versions = re.findall('([^-<>]+).tar.gz', content)
        return versions[-1]
    except urllib.error.HTTPError:
        return None
```

## Update Methods

We will use pip to update the package, here is the implementation.

```python
import os
import sys

def get_python_cmd():
    """Get current running python executable"""
    return sys.executable


def update_pkg(name, *args):
    """Update a pypi package with pip."""

    arguments = [get_python_cmd(), '-m pip install -U', name]
    arguments.extend(args)
    cmd = ' '.join(arguments)

    print('Update {}: \n{}\n'.format(name, cmd))
    os.system(cmd)
```

The update method leaves `*args` is for internal package, you might want to parse in `--extra-index-url` and `--trust-host`.

## Combine Together

Finally, we implement the auto update method.

```python
# auto_update.py
import os

from utility import get_pkg_version, get_latest_version, update_pkg

NAME = 'your_pkg_name'

def check_update(install=True):
    """Check update for the package."""
    latest_version = get_latest_version(NAME)
    installed_version = get_pkg_version(NAME)
    
    if latest_version is None or install_version is None:
        return

    if latest_version != installed_version:

        if install:
            update_pkg(NAME)
        else:
            print('New version of {} is available, {}=>{}.'.
                  format(NAME, installed_version, latest_version))

```

Then, we place it in `__init__.py` in root of package, every time the package be imported, it will run the auto checker.

```python
# your_pkg/__init__.py
from auto_update import check_update

check_update()
```

OK, we have done the auto update.

## Can Be Improved

In above example, we did not reload the module if there is an update, that depends on you. 

Another thing can be improved is, we might not want to update the pacakge when there is an update, we just want to update it if there is a major / breaking update, the updator should be smarter.