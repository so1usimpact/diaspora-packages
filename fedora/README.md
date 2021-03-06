## Diaspora RPM tools

Provides classic, binary RPM packages for deployment on Fedora.

#### Prerequisites:

- For Fedora 13 ruby-1.8, rubygem, git  and rake as described in
  [RPM installation Fedora](http://github.com/diaspora/diaspora/wiki/Rpm-installation-on-fedora)
  or [Installing-on-CentOS-Fedora](http://github.com/diaspora/diaspora/wiki/Installing-on-CentOS-Fedora).

- For Fedora 14: install required ruby packages:
    *yum install ruby ruby-ri ruby-rdoc ruby-devel rubygems git rake*

- A personal environment to build RPM:s, also described in
  [RPM installation Fedora](http://github.com/diaspora/diaspora/wiki/Rpm-installation-on-fedora)

#### Building rpm:s

Install g++ and gcc:
    % yum install gcc-c++

Bootstrap the distribution from git:
    % git clone git://github.com/diaspora/diaspora-packages.git
    % cd diaspora-packages

Create and  the diaspora bundle and application tarballs in
diaspora-packages/source/dist according to [source README]
(http://github.com/diaspora/diaspora-packages/tree/master/source)
    % cd source
    % ./make-dist.sh source
    % ./make-dist.sh bundle

Setup links from  tarballs to RPM source directory and create spec files:
    % cd ../fedora
    % ./prepare-rpm.sh

Build rpms:
    rpmbuild -ba dist/diaspora.spec
    rpmbuild -ba dist/diaspora-bundle.spec

#### Installing and running rpm:s

Install (as root, the exact name of your rpm will differ):
    rpm -U ~/rmpbuild/rpms/i686/diaspora-bundle-rt-0.0-1.1010042345_4343fade43.fc13.i686
    rpm -U ~/rmpbuild/rpms/noarch/diaspora-0.0-1.1010042345_4343fade43.fc13.noarch

Initiate (as root).
    /usr/share/diaspora/diaspora-setup

Start development server:
    sudo
    su - diaspora
    cd /usr/share/diaspora/master
    ./script/server

Start production server:
    service diaspora start

See [Using Apache](http://github.com/diaspora/diaspora/wiki/Using-apache) for
apache/passenger setup. After configuration, start with:
    initctl start diaspora-redis
    initctl start diaspora-websocket
    initctl start diaspora-resque
    /sbin/service httpd restart

#### Notes

Services are implemented using upstart with files /etc/init/diaspora-\* To control
separate services:
    initctl <status|start|stop> diaspora-thin
    initctl <status|start|stop> diaspora-websocket
    initctl <status|start|stop> diaspora-redis
    initctl <status|start|stop> diaspora-resque

All services are controlled by the SysV /etc/init.d/diaspora script. To
start/stop all services:
    service diaspora start
    service diaspora stop

prepare-rpm.sh prepare creates links  also for all files listed in SOURCES.
Typically, this is  secondary sources listed in the spec file.

Above steps creates and uses a runtime (-rt) bundle which not contains the
test or development tools. To build and install a bundle containing all
of this stuff (exact filenames varies):
    rpmbuild -ba --with dev dist/diaspora-bundle.spec
    rpm -U ~/rmpbuild/rpms/i686/diaspora-bundle-dev-0.0-1.1010042345_4343fade43.fc13.i686

The spec-files in dist/ are patched by *./prepare-rpm.sh* to reference
correct versions of diaspora and diaspora-bundle.  Editing spec files should be
done in this directory, changes in dist/ are lost when doing *./prepare-rpm.sh *.

The topmost comment's version is patched to reflect the complete version
of current specfile .  Write the comment in this directory, copy-paste
previous version nr. It will be updated.

This has been confirmed to start up and provide basic functionality both using
the thin webserver and apache passenger, on 32/64 bit systems and in the
mock build environment. Irregular nightly builds are available form time to time
at [ftp://mumin.dnsalias.net/pub/leamas/diaspora/builds](ftp://mumin.dnsalias.net/pub/leamas/diaspora/builds)

#### Implementation

Diaspora files are stored in /usr/share/diaspora, and owned by root. The
bundle, containing some C extensions, is architecture-dependent and lives
in /usr/lib[64]/diaspora. Log files are in /var/log/diaspora. Symlinks in
/usr/share diaspora makes log, bundle  and tmp dir available as expected by
diaspora app.  This is more or less as mandated by LSB and Fedora packaging rules.
To find out the actual situatiuon:

    find . -type l -exec ls -l {} \; | awk '{print $9, $10, $11}'


#### Discussion

The 1.8.7 rebuild in Fedora 13 is a pain.

For better or worse, this installation differs from the procedure outlined
in the original README.md:

- All configuration is done in /usr/share/diaspora. No global or user
  installed bundles are involved. Easier to maintain, but a mess if there
  should be parallel installations.

- A more secure setup: Service runs under it's own uid, write-protected
  installation, reasonable permissions on the mongo data files (as
  opposed to 777).

- Splitting in two packages makes sense IMHO. The bundle is not changed
  that often, but is quite big: ~35M without test packages (the default) or
  ~55M with test packages. The application is just ~3M, and is fast to
  deploy even with these tools (yes, I know, capistrano is much faster...)

- Many, roughly 50% of the packages in the bundle are already packaged
  for Fedora i. e., they could be removed from the bundle and added as
  dependencies instead.  This is likely to make things more stable in the
  long run.  fedora/diaspora.spec has a list.


