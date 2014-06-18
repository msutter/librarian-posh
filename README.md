Librarian-Posh
==============

Librarian-Posh is a bundler for your Powershell Poshfile based infrastructure repositories.
It's a fork of the librarian-chef project.

It is a tool that helps you manage your Powershell Poshfile .
Here are some more details.

Librarian-Posh is a bundler for infrastructure repositories. You can
use Librarian-Posh to resolve your infrastructure's powershell Poshfile dependencies, fetch
them, and install them into your infrastructure repository.

Librarian-Posh can resolve and fetch third-party, publicly-released powershell Poshfile,
and install them into your infrastructure repository. It can also source
powershell Poshfile directly from their own source control repositories.

Librarian-Posh can also deal with powershell Poshfile you may actively be working on
outside your infrastructure repository. For example, it can deal with powershell Poshfile
directly from their own private source control repositories, whether they are
remote or local to your machine, and it can deal with powershell Poshfile released to and
hosted on a private powershell Poshfile server.

Librarian-Posh is not primarily intended for dealing with the powershell Poshfile you are
actively working on *within* your infrastructure repository. In such a case, you
can still use Librarian-Posh, but it is likely unnecessary.

Librarian-Posh *takes over* your `posh_modules/` directory and manages it for you
based on your `Poshfile`. Your `Poshfile` becomes the authoritative source for
the powershell Poshfile your infrastructure repository depends on. You should not modify
the contents of your `posh_modules/` directory when using Librarian-Posh.

### The Poshfile

Every infrastructure repository that uses Librarian-Posh will have a file named
`Poshfile` in the root directory of that repository. The full specification for
which third-party, publicly-released powershell Poshfile your infrastructure repository
depends will go here.

Here's an example `Poshfile`:

    site "https://nuget.org/api/v2"

    mod "pester"
    mod "psake", "0.0.1"

    mod "PSDesiredStateConfigurationMissingMofs",
      :git => "https://github.com/msutter/PSDesiredStateConfigurationMissingMofs.git",
      :ref => "v0.0.1"

    mod "DSC",
      :git => "https://github.com/PowerShellOrg/DSC.git",
      :path => "Resources/cComputerManagement"

Here's how it works:

We start off by declaring the *default source* for this `Poshfile`.

    site "https://nuget.org/api/v2/"

This default source in this example is the NuGet Gallery Site API. This is
most likely what you will want for your default source. However, you can
certainly set up your own API-compatible Nuget endpoint if you want more control.

Any time we declare a powershell module dependency without also declaring a source for
that powershell module dependency, Librarian-Posh assumes we want it to look for that
powershell module in the default source.

Any time we declare a powershell module dependency that has subsidiary powershell module
dependencies of its own, Librarian-Posh assumes we want it to look for the
subsidiary powershell module dependencies in the default source.

    mod "pester"

Our infrastructure repository depends on the `pester` powershell module from the default
source. Any version of the `pester` powershell module will fulfill our requirements.

     mod "psake", "0.0.1"

Our infrastructure repository depends on the `psake` powershell module from the
default source. But only version `0.0.1 ` of that powershell module will do.

     mod "PSDesiredStateConfigurationMissingMofs",
      :git => "https://github.com/msutter/PSDesiredStateConfigurationMissingMofs.git",
      :ref => "v0.0.1"

Our infrastructure repository depends on the `PSDesiredStateConfigurationMissingMofs` powershell module,
but not the one from the default source. Instead, the powershell module is to be fetched from the
specified Git repository and from the specified Git tag only.

When using a Git source, we do not have to use a `:ref =>`. If we do not,
then Librarian-Posh will assume we meant the `master` branch.

If we use a `:ref =>`, we can use anything that Git will recognize as a ref.
This includes any branch name, tag name, SHA, or SHA unique prefix. If we use a
branch, we can later ask Librarian-Posh to update the powershell module by fetching the
most recent version of the powershell module from that same branch.

The Git source also supports a `:path =>` option. If we use the path option,
Librarian-Posh will navigate down into the Git repository and only use the
specified subdirectory. Many people have the habit of having a single repository
with many powershell modules in it. If we need a powershell module from such a repository, we can
use the `:path =>` option here to help Librarian-Posh drill down and find the
powershell module subdirectory.

   mod "cComputerManagement",
      :git  => "https://github.com/PowerShellOrg/DSC.git",
      :path => "Resources/cComputerManagement"

Our infrastructure repository depends on the `cComputerManagement` powershell module, which we have
downloaded and copied into our repository. In this example, `Resources/cComputerManagement`
is only for use with Librarian-Posh. Librarian-Posh will, instead, copy this powershell module from where
we vendored it in our repository into the `posh_modules` directory for us.

The `:path =>` source won't be confused with the `:git =>` source's `:path =>`
option.

Also, there is shortcut for powershell Poshfile hosted on GitHub, so we may write:

    mod "xPSDesiredStateConfiguration",
      :github => "msutter/xPSDesiredStateConfiguration"

### How to Use

Install Librarian-Posh:

    $ gem install Librarian-Posh

Prepare your infrastructure repository:

    $ cd ~/path/to/repo
    $ echo /posh_modules >> .gitignore
    $ echo /tmp >> .gitignore

Librarian-Posh takes over your `posh_modules/` directory, and will always reinstall
the powershell modules listed in the `Poshfile.lock` into your `posh_modules/` directory. Hence
you do not need your `posh_modules/` directory to be tracked in Git. If you
nevertheless want your `posh_modules/` directory to be tracked in Git, simply don't
`.gitignore` the directory.

Librarian-Posh uses your `tmp/` directory for tempfiles and caches. You do not
need to track this directory in Git.

Make a Poshfile:

    $ librarian-posh init

This creates an empty `Poshfile` with the NuGet Gallery Site API as the
default source.

Add dependencies and their sources to the `Poshfile`:

    $ cat Poshfile
        site "https://nuget.org/api/v2"
        mod "pester"
        mod "psake", "0.0.1"
        mod "PSDesiredStateConfigurationMissingMofs",
          :git => "https://github.com/msutter/PSDesiredStateConfigurationMissingMofs.git",
          :ref => "v0.0.1"
        mod "DSC",
          :git => "https://github.com/PowerShellOrg/DSC.git",
          :path => "Resources/cComputerManagement"


This is the same `Poshfile` we saw above.

    $ librarian-posh install [--clean] [--verbose]

This command looks at each `mod` declaration and fetches the powershell module from
the source specified, or from the default source if none is provided.

Each powershell module is inspected, its dependencies are determined, and each dependency
is also fetched. For example, if you declare `mod 'poweryaml'`, which
depends on other powershell modules such as `'pester'`, then those other powershell modules
including `'pester'` will be fetched. This goes all the way down the chain of
dependencies.

This command writes the complete resolution into `Poshfile.lock`.

This command then copies all of the fetched powershell Poshfile into your `posh_modules/`
directory, overwriting whatever was there before.

Check your `Poshfile` and `Poshfile.lock` into version control:

    $ git add Poshfile
    $ git add Poshfile.lock
    $ git commit -m "I want these particular versions of these particular powershell modules."

Make sure you check your `Poshfile.lock` into version control. This will ensure
dependencies do not need to be resolved every run, greatly reducing dependency
resolution time.

Get an overview of your `Poshfile.lock` with:

    $ Librarian-Posh show

Inspect the details of specific resolved dependencies with:

    $ Librarian-Posh show NAME1 [NAME2, ...]

Update your `Poshfile` with new/changed/removed constraints/sources/dependencies:

    $ cat Poshfile
        site "https://nuget.org/api/v2"
        mod "pester"
        mod "psake", "0.0.1"
        mod "PSDesiredStateConfigurationMissingMofs",
          :git => "https://github.com/msutter/PSDesiredStateConfigurationMissingMofs.git",
          :ref => "v0.0.1"
        mod "cComputerManagement",
          :git => "https://github.com/PowerShellOrg/DSC.git",
          :path => "Resources/cComputerManagement"
        mod 'xWebAdmin' # new!
    $ git diff Poshfile
    $ librarian-posh install [--verbose]
    $ git diff Poshfile.lock
    $ git add Poshfile
    $ git add Poshfile.lock
    $ git commit -m "I also want these additional powershell modules."

Find out which dependencies are outdated and may be updated:

    $ librarian-posh outdated [--verbose]

Update the version of a dependency:

    $ librarian-posh update pester psake cComputerManagement [--verbose]
    $ git diff Poshfile.lock
    $ git add Poshfile.lock
    $ git commit -m "I want updated versions of these powershell modules."

Push your changes to the git repository:

    $ git push origin master

### Configuration

Configuration comes from three sources with the following highest-to-lowest
precedence:

* The local config (`./.librarian/posh/config`)
* The environment
* The global config (`~/.librarian/posh/config`)

You can inspect the final configuration with:

    $ librarian-posh config

You can find out where a particular key is set with:

    $ librarian-posh config KEY

You can set a key at the global level with:

    $ librarian-posh config KEY VALUE --global

And remove it with:

    $ librarian-posh config KEY --global --delete

You can set a key at the local level with:

    $ librarian-posh config KEY VALUE --local

And remove it with:

    $ librarian-posh config KEY --local --delete

You cannot set or delete environment-level config keys with the CLI.

Configuration set at either the global or local level will affect subsequent
invocations of `librarian-posh`. Configurations set at the environment level are
not saved and will not affect subsequent invocations of `librarian-posh`.

You can pass a config at the environment level by taking the original config key
and transforming it: replace hyphens (`-`) with underscores (`_`) and periods
(`.`) with doubled underscores (`__`), uppercase, and finally prefix with
`LIBRARIAN_POSH_`. For example, to pass a config in the environment for the key
`part-one.part-two`, set the environment variable
`LIBRARIAN_POSH_PART_ONE__PART_TWO`.

Configuration affects how various commands operate.

* The `path` config sets the powershell modules directory to install to. If a relative
  path, it is relative to the directory containing the `Poshfile`. The
  equivalent environment variable is `LIBRARIAN_POSH_PATH`.

* The `tmp` config sets the cache directory for librarian. If a relative
  path, it is relative to the directory containing the `Poshfile`. The
  equivalent environment variable is `LIBRARIAN_POSH_TMP`.

* The `install.strip-dot-git` config causes the `.git/` directory to be stripped
  out when installing powershell Poshfile from a git source. This must be set to exactly
  "1" to cause this behavior. The equivalent environment variable is
  `LIBRARIAN_POSH_INSTALL__STRIP_DOT_GIT`.

Configuration can be set by passing specific options to other commands.

* The `path` config can be set at the local level by passing the `--path` option
  to the `install` command. It can be unset at the local level by passing the
  `--no-path` option to the `install` command. Note that if this is set at the
  environment or global level then, even if `--no-path` is given as an option,
  the environment or global config will be used.

* The `install.strip-dot-git` config can be set at the local level by passing
  the `--strip-dot-git` option to the `install` command. It can be unset at the
  local level by passing the `--no-strip-dot-git` option.

How to Contribute
-----------------

### Running the tests

Ensure the gem dependencies are installed:

    $ bundle

Run the tests

    $ [bundle exec] rspec spec

### Installing locally

Ensure the gem dependencies are installed:

    $ bundle

Install from the repository:

    $ [bundle exec] rake install

### Reporting Issues

Please include relevant `Poshfile` and `Poshfile.lock` files. Please run the
`librarian-posh` commands in verbose mode by using the `--verbose` flag, and
include the verbose output in the bug report as well.

License
-------

Forked from Jay Feldblum's librarian-chef.
Adapted to posh by Marc Sutter

Copyright (c) 2013 ApplicationsOnline, LLC.

Released under the terms of the MIT License. For further information, please see
the file `LICENSE.txt`.
