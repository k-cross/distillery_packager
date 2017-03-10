# Distillery Packager

Elixir lib for creating linux packages with Distillery.

## Functionalities

 1. [x] Able to build debian packages:
     1. [x] With changelog
     2. [x] With control file
 2. [x] Ability to add in pre/post install/remove scripts
 3. [x] Validates configuration before completing the build
 4. [x] Ability for you to replace file templates with your own
 5. [x] Ability to manage additional files for package
 6. [x] Automatically builds init scripts:
     1. [x] Systemd
     2. [x] Upstart
     3. [x] SysVinit

## Required OS dependencies

Before using distillery_packager, you'll need the following packages installed and in your path:

 - `tar` (or `gtar` if you're on a mac - you can `brew install gnu-tar` if you don't already have it)
 - `ar`
 - `uname`

## General configuration

Distillery_packager relies on the following data in the `mix.exs` file being set:

```diff
defmodule Testapp.Mixfile do
   use Mix.Project

   def project do
      [app: :testapp,
      version: "0.0.1",
      elixir: "~> 1.4",
+     description: "Elixir lib for creating linux packages with Distillery",
      build_embedded: Mix.env == :prod,
      start_permanent: Mix.env == :prod,
-     deps: deps()]
+     deps: deps(),
+     deb_package: deb_package()]
   end
```

## Debian package configuration

The `deb_package` function must be set as:

```diff
def deb_package do
   [
      vendor: "18Months S.r.l.",
      maintainers: ["18Months <info@18months.it>"],
      homepage: "https://www.18months.it",
      licenses: ["MIT"],
      external_dependencies: [],
      maintainer_scripts: [
         pre_install: "rel/distillery_packager/debian/install_scripts/pre_install.sh",
         post_install: "rel/distillery_packager/debian/install_scripts/post_install.sh",
         pre_uninstall: "rel/distillery_packager/debian/install_scripts/pre_uninstall.sh"
      ]
      config_files: ["/etc/init/.conf"],
      additional_files: [{"configs/config1", "etc/distillery_packager/config/"},
                         {"configs/config2", "etc/distillery_packager/config/"}]
      owner: [user: "root", group: "root"]
   ]
end
```

A list of configuration options you can add to `deb_package/0`:

 - `vendor`
   - String
   - The distribution vendor that's creating the debian package. I normally just put my name or company name.
 - `maintainers`
   - Array of Strings
   - Should be in the format `name <email>`
 - `homepage`
   - String
   - Should be in the format `https://www.18months.it`
 - `licenses`
   - Array of Strings
   - Can be something like `["Copyright <date> <company_name>"]` if you are building private packages.
 - `external_dependencies`
   - Array of Strings
   - Should be in the format of `package-name (operator version_number)` where operator is either `<<`, `<=`, `=`, `>=`, or `>>` - [read more about this here.][1]
 - `maintainer_scripts`
   - A keyword list of Strings
   - The keyword should be one of: `:pre_install`, `:post_install`, `:pre_uninstall`, or `:post_uninstall`
   - The keyword should point to the path of a script you want to run at the moment in question.
 - `config_file`
   - Array of Strings
   - Should contain the absolute path of the config file to be overwritten.
 - `additional_files`
   - List of Tuples
   - Should contain the relative path of the source file to copy in the first position of the tuple. Path is relative to rel/distillery_packager/additional_files. It is mandatory to create this path, if you want to use this feature.
   - Should contain the absolute path of the destination folder, where copy the file, in the second position of the tuple
 - `owner`
   - A keyword list of Strings
   - If set, requires both `user` and `group` keys to be set.
   - This is used when building the archive to set the correct user and group
   - Defaults to root for user & group.

## Distillery configuration

You can build a deb by adding `plugin DistilleryPackager.Plugin` to your `rel/config.exs` file.

You can also specify target distribution with `plugin DistilleryPackager.Plugin, distribution: "xenial"`

The name and version is taken from the `rel/config.exs` file.

## Installation

The package can be installed as:

  1. Add distillery_packager to your list of dependencies in `mix.exs`:

        def deps do
          [{:distillery_packager, "~> 0.3"}]
        end

## Usage

### Customising deb config files

You can customise the debs that are being built by copying the template files used and modifying them:

```bash
mix release.deb.generate_templates
```

### Build

Packages are build with `mix release` command.


[1]:https://www.debian.org/doc/manuals/maint-guide/dreq.en.html#control
