## Personal Vagrant environment to configure one PE master and x agents.

If using [puppetlabs Vagrant boxes](https://atlas.hashicorp.com/puppetlabs), it
calculates OS, version and architecture to download the right PE installer.

Fill the following variables in Vagrantfile:

* check_update: Vagrant checks for new versions of the box image. False to disable. True is the default.
* iprange: What subnet your boxes are going to be in
* masterip: IP for the master. I go with $iprange+'.50'
* domain: Boxes get named as master.#{domain} and agentx.#{domain}.
* startip: The starting IP for the agents.
* box: Shortname of the Vagrant box to use. Notice both master and agents use the same box.
* arch: Architecture of the boxes to select the right puppet installer. Ex: 'el-6-x86_64'
* puppetver: Puppet Enterprise version, defaults to "latest". Ex: '2015.3.3'
* agents: How many agents to create. Set to 0 to not create agents.
* install_gitlab: Do you want a gitlab server?

TODO:
* bash -> ruby
