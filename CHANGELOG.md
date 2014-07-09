ginas changelog
===============

## July 2014

`postgresql` role has been modified to support more granular auth parameters in
`pg_hba.conf` configuration files, thanks to [Nick Janetakis](https://github.com/nickjj)
and welcome to the team!

`etc_services` role can now add custom service entries using lists in inventory
and dependency variables.


## June 2014

After some hiatus, time to go back to work!

### LXC

Huge Linux Containers rewrite, which brings LXC 1.0 support to Debian Wheezy.

Since verion 1.0 will probably not land in Wheezy itself, 'lxc' role will
automatically build and install LXC 1.0 Debian packages on Debian Wheezy (this
will not happen on other distributions or suites). Additionally, 'lxc' .deb
packages will be made available to the ginas cluster via "reprepro" repository
(built packages will be copied to special directory on Ansible Controller where
'reprepro' role expects .deb packages which then will be automatically
downloaded to local apt repository on next Ansible run). This way, you can
build LXC 1.0 packages on one spare host, and then have them available all the
time for all other hosts without any -dev packages required.

Because proper LXC support requires newer Linux kernel than the one available
in Debian Wheezy by default, 'lxc' role will install current Linux kernel from
`wheezy-backports` repository and will send an e-mail to administrator about
required server reboot. 'lxc' role will not configure and manage containers
without that reboot (idempotence is still maintained within the role).

Default container generation has been changed from `multistrap` to
`debootstrap`, which is used by modified `lxc-debian` script called `lxc-ginas`
- script will automatically install packages required by Ansible and configure
default administrator account with SSH key from Ansible Controller and
`sudo` access.

You can now manage Linux containers using a simple YAML list, with options to
start, stop or destroy containers, select different container templates, etc.
See `playbooks/roles/ginas.lxc/defaults/main.yml` file for more information.

### Postfix and Mailman

Mailman support in 'postfix' role has been removed. Instead, 'postfix' role
gained a set of new `postfix_dependent_*` variables which can be used to
configure parts of Postfix configuration (`/etc/postfix/main.cf` and
`/etc/postfix/master.cf` files) from other roles using dependency variables.
Those changes are idempotent and are saved in Ansible facts by 'postfix' role.

Examples of new functionality can be found in
`playbooks/roles/ginas.postfix/defaults/main.yml` file. Mailman role also is
now configured to use these variables, you can check new configuration in
`playbooks/roles/ginas.mailman/meta/main.yml` to see the details.

### Other news

`contrib/bootstrap-ansible.sh` script has been updated to work with new `make
deb` output. Ansible now requires `build-essential` and `devscripts` packages
to create .deb packages correctly.

After some changes to variables in 'apt' role, APT configuration was not able
to use `apt-cacher-ng` automatically. Now cache will be used correctly.


## May 2014

A definition of "public API" has been added in CONTRIBUTING.md. Following that,
ginas will start using git tags for stable releases. This changelog will change
format from monthly to version-based on release of v0.0.0.

All ginas roles have been renamed from "rolename" to "ginas.rolename", with
updated dependencies, to make them similar to roles downloaded from Ansible
Galaxy. This should help you use ginas roles in your own environment and
playbooks, using `roles_path` variable in Ansible.

### New roles

- **nodejs**: [NodeJS](http://nodejs.org/) is a platform built on Chrome's
  JavaScript runtime for easily building fast, scalable network applications.

- **etherpad**: [Etherpad Lite](http://etherpad.org/) is a collaborative web
  text editor written using NodeJS.

- **gitlab_ci** and **gitlab_ci_runner**: [GitLab CI](https://www.gitlab.com/gitlab-ci/)
  is a Continuous Integration server which integrates with
  [GitLab](https://www.gitlab.com/). It is divided into two components - GitLab
  CI itself which coordinates work between GitLab and its runners, and GitLab
  CI Runner which executes provided scripts.

### secrets

Support for "secret" directory on Ansible Controller has been rewritten (third
time, hopefully the last). Previously 'secret' variable usage was optional and
it needed to be defined manually by the user; now it will be required (with
time, tasks that depend on it will not check if 'secret' variable is defined,
to simplify the code) and will be automatically defined relative to currently
selected inventory. There are several variables that can be used to influence
this behaviour. More information and explanation of the concept can be found in
[README.md](https://github.com/ginas/ginas/blob/master/playbooks/roles/secret/README.md)
file of the 'secret' role.

To further redesign the 'secret' concept, automatic encryption of the directory
using `encfs` has been disabled. If you currently have encrypted secret
directory and want to keep the contents, you should decrypt it and move the
contents to the new, unencrypted, secret directory. To protect the data
I suggest using an encrypted filesystem, like eCryptfs or LUKS.

There are currently no plans to enable encryption for secret directory, but it
might happen in the future. At the moment users should secure the data by
themselves. Any suggestions how to bring back encryption in a reliable and easy
way are welcome.

### nginx

nginx installation from Debian Backports has been temporarily disabled due to
problems with first daemon startup on install - when nginx from backports is
installed, an empty pidfile is created on first startup and system init script
does not have control over nginx daemon, which results in an error when nginx
needs to be restarted. When this error is resolved, change will be reverted.

You can now tell nginx role to use IP-based SSL certificates (instead of
FQDN-based), useful for hosts without DNS domain name, like Vagrant virtual
machines.

You can disable 'favicon.ico' filtering in nginx server configuration and let
your application handle the icon (used by Etherpad role).

### GitLab

`gitlab` role has been updated with support for GitLab CE 6.9. This update is
very light, without additional set of config files, because changes made in
'6-9-stable' branch added only commented out code which means that older config
files should work fine.

`gitlab-shell` will no longer use a separate nginx server instance to
communicate with GitLab, instead it will talk to unicorn server directly.
Unicorn gained a port variable and is now registered as a separate service in
`/etc/services`.

### Other news

Default Vagrant setup has been simplified to use only one server with
a selection of web applications to install (either GitLab, ownCloud or phpIPAM,
selectable via Vagrant inventory).

If you use Debian Preseed server, you can define short hostnames in DNS and use
them instead of long hostname to point installer to a correct preseed file
- for example instead of using `url=destroy.debian.nat.example.org`, you can
use `url=destroy.debian`. You can also change the domain that is configured for
Debian Preseed server, when for example your datacenter uses long hostnames for
its servers.

`interfaces` role has been modified to use separate variable for interface list
defined via dependency. This will allow interfaces to be configured in
subsequent plays instead of just the common one (needed for example by `nat`
role to configure separate bridge interface).

Abusive Hosts Blocking List has been removed from Postfix DNSBL list because of
[impending end of the service](http://ahbl.org/content/changes-ahbl).

Travis CI build has been modified to test idempotence of the playbook - it is
run a second time to check if there are any changes.


## April 2014

### New roles

- **dhcpd**: [ISC DHCP Server](https://www.isc.org/downloads/dhcp/) with
  extensive configuration template that accounts for multiple networks, and
  nesting. Non-authoritative by default, requires configuration by the admin.

- **mailman**: [GNU Mailing List Manager](http://www.list.org/), integrated
  with Postfix. Role allows you to create or remove multiple mailing lists,
  other configuration is done using web interface.

- **ntp**: [OpenNTPd](http://www.openntpd.org/), previously installed by `apt`
  role, has been moved to it's own role, with separate configuration. It will be
  installed on all hosts, excluding OpenVZ/LXC containers.

- **openvz**: [OpenVZ](http://openvz.org/Main_Page) support is based on
  packages from Debian Wheezy and custom kernel from `openvz.org` repository.
  Role will automatically configure container migration over SSH when multiple
  OpenVZ Hardware Nodes are configured, either as one big cluster, or multiple
  separate clusters.

### GitLab

In a span of a month, `gitlab` role gained support for two new releases
(6.7 and 6.8) and lost support for 6.5 and 6.6 releases due to [removal
of modernizr gem](https://github.com/gitlabhq/gitlabhq/issues/6687) which
prevents automated installation of older GitLab releases. Since GitLab 6.8 has
been officially released, upgrade from older release is unaffected by the
change and can still be tested.

GitLab role gained support for PostgreSQL database (both installation and
upgrades). Currently `postgresql` role in ginas is not very secure and does
not support creation of roles and databases using dependency variables
(similar to MySQL). Fixes are planned in the future. For now, production
GitLab instances should use MySQL database.

### nginx

You can install nginx package with support for SPDY and OCSP stapling from
Debian Backports (it is enabled by default via APT pinning in `apt` role).
Currently nginx package from Jessie seems to have problems with pid file which
is not created correctly on first install and system cannot stop or restart
`nginx` daemon. Killing the daemon manually and restarting `nginx` fixes the
problem.

nginx role gained support for Diffie-Hellman parameter generation (by default
done once a day via a cron script). Because of that, `openssl` command is now
a requirement, which forced `pki` role to be run almost at the beginning of
'common.yml' playbook.

Lists of available SSL cipher suites have been updated, and now you can also
see the source for a particular list (as a comment). 'nginx' role allows you
to select a particular ECDH curve, either globally for a host, or for a server
instance.

You can now define a list of allowed referer hosts which can hotlink
a particular location. This function was added to block access to Mailman web
interface from domains other than the allowed ones, which should prevent
subscribe backscatter spam via forms hosted on external servers.

### Other news

I've added `CONTRIBUTING.md` text file with information about project, mailing
list and IRC channel.

`CHANGELOG.md` file has been added which will describe significant changes in
ginas on a monthly basis.
