# DynamicHosts
Control various host configurations dynamically

## Overview ##
I experiment a lot with both virtual hosts and network connected microcontrollers. I also have 
Naemon monitoring things and it is a real pain to have to move configuration files around so
there are no spurious alerts being generated when I disconnect something. In addition, I like to have
"real" host names defined in dnsmasq.

Using an idea from the Linux Apache configuration scheme, this project uses two directories to
dynamically configure applications with hosts. It only requires one change to the config file.

Add an additional line in naemon.cfg to point to an additional directory called "available.hosts"
and create two directories: available.hosts and enabled.hosts.

Likewise, a line in dnsmasq.conf will direct dnsmasq to monitor a directory for changes.

Available.hosts contains host configuration files that can be enabled. Enabled.hosts contains
symbolic links to available.hosts and are hosts that are active.


## Structure ##
The whole process is contained in Python files called monitor-manage and dns-manage. There are three symbolic links
that point to the main file: xxx-enable, xxx-disable, and xxx-list. By using argv[0], the manager file can determine which
command was entered and thus execute the proper actions. The only requirement is that the manager must
have sufficient rights to modify enabled.hosts and be able to execute systemctl to restart Naemon.

At this time, the program is rather crude and doesn't have a lot of checking, but that can be added
without too much effort. One big problem is that if the host configuration file fails, Naemon will not
start. So some serious testing of the configuration is suggested before trusting the process.

## Installation ##
This was tested using an Ubuntu 16.04 server with the default Naemon and dnsmasq installations. For other
distributions, you may have to modify paths or file names. I did not test this with Nagios, but
it should work if the paths are changed to match the installation.

For Naemon:
* Add cfg_dir=/etc/naemon/hosts.enabled to naemon.cfg.
* Add two directories to /etc/naemon named hosts.available and hosts.enabled.
* Copy monitor-manage to /usr/bin. Make sure it is executable.
* Create three symbolic links to modhost in /usr/bin named monitor-enable, monitor-disable,monitor-list.
* Put your host configuration files in hosts.available.

For dnsmasq:
* Add hostsdir=/etc/dnsmasq.d/hosts.enabled to dnsmasq.conf.
* Add two directories to /etc/dnsmasq.d named hosts.available and hosts.enabled.
* Copy dns-manage to /usr/bin. Make sure it is executable.
* Create three symbolic links to dns-manage in /usr/bin named dns-enable, dns-disable, dns-list.
* Put your host configuration files in available.hosts.

## Usage ##
Use xxx-list to see what hosts are defined. It will show those in both available and enabled.
To enable a host, just execute xxx-enable <hostname>. 
To disable a host, just execute xxx-disable <hostname>.

My version of the program embeds the ".dns" suffix in the code so you just use the host name
in the command line, not the "host.dns". You can change this to match your schema.

The program does basic checking of the files to insure the existence of an available config
and enabled  config so you can't change a working configuration.
