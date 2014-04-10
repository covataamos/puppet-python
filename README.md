python
======

This Puppet module installs the Python language runtime, and provides classes
for common Python packages and tools.  In addition, an improved pip package
provider ([`pipx`](#pipx)) is included as well as custom types for Python
virtual environments ([`venv`](#venv)) and packages inside virtual environments
([`venv_package`](#venv_package)).


By default, including the `python` class installs Python, setuptools, and pip;
the `python::virtualenv` class installs virtualenv.  Thus, to have Python, pip,
and virtualenv installed on your system simply place the following in your
Puppet manifest:

```puppet
include python
include python::virtualenv
```

This module supports Debian, RedHat, OpenBSD, Solaris, and Windows platforms
-- Windows users should read the [Windows Notes](#windows-notes).

Python Classes
--------------

### `python::devel`

Instals the Python development headers package for the system, useful
if you need to install packages with C extensions with `pip`.  For
example, to install [PyCrypto](https://www.dlitz.net/software/pycrypto/)
you could use the following (assuming a non-Windows platform):

```puppet
include python
include python::devel

package { 'pycrypto':
  ensure   => installed,
  provider => 'pip',
  require  => Class['python::devel'],
}
```

### `python::django`

Installs Django in the system site-packages using `pip`.

```puppet
include python
include python::django
```

### `python::flask`

Installs Flask in the system site-packages using `pip`, for example:

```puppet
include python
include python::flask
```

### `python::virtualenv`

Installs [virtualenv](http://www.virtualenv.org), using the default system
package.  If the system package is too old for your taste, tell it to be
installed using `pip` instead, for example:

```puppet
include python
class { 'python::virtualenv':
  provider => 'pip',
}
```

Python Types
------------

### `pipx`


The `pipx` package provider is an enhanced version of Puppet's own
[`pip`](http://docs.puppetlabs.com/references/latest/type.html#package-provider-pip)
provider.  It includes such enhancements as:

* Implements the [`install_options`](http://docs.puppetlabs.com/references/latest/type.html#package-attribute-install-options) feature,
  where you may specify the [pip install options](http://pip.readthedocs.org/en/latest/reference/pip_install.html#options).
* Improvements for installing packages from version control
* Uses HTTPS to query PyPI when setting [`ensure`](http://docs.puppetlabs.com/references/latest/type.html#package-attribute-ensure) to 'latest'

For example, assuming you had an internal PyPI mirror at
`https://pypi.mycorp.com`, you could install the `requests` package system-wide
from your mirror using the following:

```puppet
package { 'requests':
  ensure          => installed,
  provider        => 'pipx',
  install_options => [
    { '--index-url' => 'https://pypi.mycorp.com' },
  ],
}
```

### `venv`

The `venv` type enables the management of Python virtual environments.
The name of the `venv` resource is the path to the virtual environment
-- for example to have your virtualenv in `/srv/venv`, you'd use:

```puppet
# Python and virtualenv are required to use `venv` type.
include python
include python::virtualenv

# Creating a virtualenv in /srv/venv.
venv { '/srv/venv': }
```

To have the virtualenv be owned by a user other than the one running
Puppet (typically `root`), you can set the `owner` and `group` parameters
(these are not supported on Windows):

```puppet
venv { '/srv/venv':
  owner => 'justin',
  group => 'users',
}
```

To have the virtualenv include the system site packages:

```puppet
venv { '/srv/venv':
  system_site_packages => true,
}
```

To delete the virtualenv from the system:

```puppet
venv { '/srv/venv':
  ensure => absent,
}
```

### `venv_package`

This type installs packages in a Python virtual environment -- the title of
a `venv_package` resource must contain the name of the package and the path
to to the virtual environment separated by the `@` symbol.  For example,
to install Django into the `venv` defined above:

```puppet
venv_package { 'Django@/srv/venv':
  ensure => installed,
}
```

The `venv` specifed after the `@` will be [automatically required](http://docs.puppetlabs.com/learning/ordering.html#autorequire).
Like the [`pipx`](#pipx) package provider, you may also specify `install_options`, e.g.:

```puppet
venv_package { 'Flask@/srv/venv':
  ensure          => installed,
  install_options => [ { '--index-url' => 'https://pypi.mycorp.com' } ],
}
```

Just like with Puppet's own `pip` provider, you can install using VCS --
for example, to install `Flask` from GitHub (at the `0.8.1` version tag):

```
include sys::git

venv_package { 'Flask@/srv/venv':
  ensure  => '0.8.1',
  source  => 'git+https://github.com/mitsuhiko/flask',
  require => Class['sys::git'],
}
```

Note: This had to be its own type (rather than a package provider)
due to the fact that there can be multiple packages on a system in
different virtual environments.

Windows Notes
-------------

Windows support requires the [`counsyl-windows`](https://github.com/counsyl/puppet-windows)
module.  Because `%Path%` updates aren't reflected in Puppet's current session,
you will see errors about not being able to find the `pip` and/or `virtualenv`
commands -- running Puppet again should make these errors go away on a fresh
system.  In addition, due to the nature of Windows platforms, customizations
should be done on the `python::windows` class before including `python`.
For example, to force the use the 32-bit version of Python 2.6.6 you would
use the following:

```puppet
class { 'python::windows':
  arch    => 'i386',
  version => '2.6.6',
}
include python
```

License
-------

Apache License, Version 2.0

Contact
-------

Justin Bronn <justin@counsyl.com>

Support
-------

Please log tickets and issues at https://github.com/counsyl/puppet-python
