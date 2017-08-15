# DynamicHosts
Control Naemon/Nagios host configurations dynamically

## Overview ##
I experiment a lot with both virtual hosts and network connected microcontrollers. I also have 
Naemon monitoring things and it is a real pain to have to move configuration files around so
there are no spurious alerts being generated when I disconnect something.

Using an idea from the Linux Apache configuration scheme, this project uses two directories to
dynamically configure Naemon with hosts. It only requires one change to the naemon.cfg file.

Add an additional line in naemon.cfg to point to an additional directory called "available.hosts"
and create two directories: available.hosts and enabled.hosts.

Available.hosts contains host configuration files that can be enabled. Enabled.hosts contains
symbolic links to available.hosts and are hosts that are being monitored.

To enable a host, just create a symbolic link and restart Naemon. To disable a host, just
delete the symbolic link and restart Naemon.

## Structure ##
The whole process is contained in a single Python file called modhost. There are three symbolic links
that point to modhost: enhost, dishost, and listhosts. By using argv[0], modhost can determine which
command was entered and thus execute the proper actions. The only requirement is that modhost must
have sufficient rights to modify enabled.hosts and be able to execute systemctl to restart Naemon.

At this time, the program is rather crude and doesn't have a lot of checking, but that can be added
without too much effort. One big problem is that if the host configuration file fails, Naemon will not
start. So some serious testing of the configuration is suggested before trusting the process.

## Installation ##
This was tested using an Ubuntu 16.04 server with the default Naemon installation. For other
distributions, you may have to modify paths or file names. I did not test this with Nagios, but
it should work if the paths are changed to match the installation.

* Add cfg_dir=/etc/naemon/hosts.enabled to naemon.cfg.
* Add two directories to /etc/naemon named available.hosts and enabled.hosts.
* Copy modhost to /usr/bin. Make sure it is executable.
* Create three symbolic links to modhost in /usr/bin named enhost, dishost,listhosts.
* Put your host configuration files in available.hosts.

## Usage ##
Use listhosts to see what hosts are defined. It will show those in both available and enabled.
To enable a host, just execute enhost <hostname>. 
To disable a host, just execute dishost <hostname>.

My version of the program embeds the ".cfg" suffix in the code so you just use the host name
in the command line, not the "host.cfg". You can change this to match your schema.

The program does basic checking of the files to insure the existence of an available config
and enabled  config so you can't change a working configuration.
