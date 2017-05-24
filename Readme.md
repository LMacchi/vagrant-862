## Personal Vagrant environment to configure one PE master and x agents.

Fill the following variables in Vagrantfile:

* check_update: Vagrant checks for new versions of the box image. False to disable. True is the default.
* domain: Boxes get named as master.#{domain} and agentx.#{domain}.
* box: Shortname of the Vagrant box to use. Notice both master and agents use the same box.
* puppetver: Puppet Enterprise version, defaults to "latest". Ex: '2015.3.3'
* agents: How many agents to create. Set to 0 to not create agents.
