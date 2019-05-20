OpenCPN plugin installer playground
===================================

The original plugin README is indeed the file README.

This is an attempt to create a non-trivial testcase for an opencpn plugin
installer, https://github.com/OpenCPN/OpenCPN/pull/1390.

In order to work with an installer, plugins must be modified. These
modifications does not hurt the current usecase; on the contrary it's
about cleaning up and update to modern standards.

The modifications can be seen as a patch series in this repo. It
includes some general clean-up:

   - Updating to the latest plugin API v 1.16 (as of 5.0.0).
   - Using the new function GetPluginDatadir instead of previous
     GetpSharedDataLocation(). This is so that the plugin can find
     it's own data directory even if relocated to a new location.
   - Ensure that LD\_LIBRARY\_PATH is not overwritten by plugin if it's
     already set.
   - Ensure that plugin does not link to libraries outside the git tree,
     in particular the msvc opencpn.lib import library.

And some new functionality:

   - Building a .tar.gz bundle together with the tradtional installer.
   - Add support for building a plugin metadata XML file.
   - Giving the package a more unique name. This is a requirement from
     deployment sites like bintray where all plugins share a common
     namespace.
   - Adding travis and appveyor continous integration support.
   - Add support for building a flatpak plugin.

Most of these changes should be trivial given the boilerplate code available
here. The last part, flatpak, is somewhat more work and also requires a
linux platform to run.

Some notes on the changes:

The *new API* is available in the directory api-16 which also contains
an msvc import library. Note that the plugin must link against these
libraries on windows (that is, msvc and mingw).

The *data directory lookup* seems to be using GetpSharedDataLocation() on
most (all?) plugins. Just searching for this and replacing it with
getPluginDataDir() should make it. Note that the signature differs, though.

Modifying *LD_LIBRARY_PATH* should be avoided if possible. If it still needs
to be done it must respect existing value, possibly adding to it.

The tar.gz package is the package used by new installer. It's just the
contents of the installation directory created by *make install*.

The XML metadata file is the basis for how plugins are made available for
installation for users. Created by cmake, simple boilerplate code.

The *CI integration* should build the plugins and deploy them together with
teh XML metadata file on a public website.

The *flatpak plugin* is the solution for non-debian linux platforms. It's
more work, but basically also boilerplate code. Making a flatpak plugin
is much easier if a version with the other fixes is available in a git repo,
upstreamed or in a fork.

The *mingw* fixes and *fedora builder* are not relevant to the general case.
mingw is anyway broken in this plugin, and the fedora builder is by no
means necessary.
