# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Vagrant environment that creates a puppet master, gitlab and $agents agents
#
#

# Variables
check_update = false
domain = 'puppetlabs.vm'
box = 'puppetlabs/centos-6.6-64-nocm'
agents = 1

## Puppet
pe_ver = "latest"
url = "https://pm.puppetlabs.com/cgi-bin/download.cgi?dist=el&rel=6&arch=x86_64&ver=#{pe_ver}"

Vagrant.configure(2) do |config|
  config.vm.box_check_update = check_update
  config.vm.define "master" do |master|
    master.vm.box = box
    master.vm.hostname = "master.#{domain}"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = "2"
    end
    master.vm.network "private_network", type: "dhcp"
    master.vm.provision "hosts" do |prov|
      prov.autoconfigure = true
    end
    master.vm.provision "shell", privileged: true, inline: <<-SHELL
      # Stop iptables
      sudo service iptables stop 2&> /dev/null && sudo chkconfig iptables off 2&> /dev/null
      if [ $? -eq 0 ]; then
        echo "IPTables stopped successfully"
      fi
      sudo /usr/local/bin/puppet --version 2&> /dev/null
      if [ $? -ne 0 ]; then
        # Download tar
        echo "Download URL: #{url}"
        echo "Downloading Puppet Enterprise, this may take a few minutes"
        sudo wget --quiet --content-disposition "#{url}"
        if [ $? -ne 0 ]; then
          echo "Puppet failed to download"
          exit 2
        fi
        # Extract tar to /root
        sudo tar xzvf /home/vagrant/puppet-enterprise-*.tar* -C /root
        if [ $? -ne 0 ]; then
          echo "Puppet failed to extract"
          exit 2
        fi
        # Add SSH config to have code manager use LMacchi github account
        if [ ! -d "/opt/puppetlabs/server/data/puppetserver/.ssh" ]; then
          sudo mkdir -p /opt/puppetlabs/server/data/puppetserver/.ssh
          sudo chmod 700 /opt/puppetlabs/server/data/puppetserver/.ssh
          sudo cp /vagrant/files/config /opt/puppetlabs/server/data/puppetserver/.ssh/config
        fi
        # Add my personal SSH keys
        if [ ! -d "/etc/puppetlabs/puppetserver/ssh" ]; then
          sudo mkdir -p /etc/puppetlabs/puppetserver/ssh
          sudo chmod 700 /etc/puppetlabs/puppetserver/ssh
          sudo cp /vagrant/puppetfiles/keys/id-control_repo.rsa* /etc/puppetlabs/puppetserver/ssh
        fi
        # Install PE from answers file
        echo "Ready to install Puppet Enterprise #{pe_ver}"
        sudo /root/puppet-enterprise-*/puppet-enterprise-installer -c /vagrant/puppetfiles/custom-pe.conf -y
        # Clean up
        sudo rm -fr /root/puppet-enterprise-*
        # Add an autosign condition
        sudo echo "*.#{domain}" > /etc/puppetlabs/puppet/autosign.conf
        # Now that pe-puppet exists, change permissions for keys and config
        sudo chown -R pe-puppet: /opt/puppetlabs/server/data/puppetserver/.ssh
        sudo chown -R pe_puppet: /etc/puppetlabs/puppetserver/ssh
        echo "Running puppet for the first time"
        sudo /usr/local/bin/puppet agent -t
        # Create deploy token with admin, so replica can be provisioned
        echo "puppetlabs" | sudo /opt/puppetlabs/bin/puppet-access login admin --lifetime 90d
        # Deploy code
        echo "Deploying puppet code from version control server"
        sudo /vagrant/scripts/deploy_code.sh
        # Clear environments cache
        echo "Clearing environments cache"
        sudo /vagrant/scripts/update_environments.sh
        # Update classes in console
        echo "Clearing classifier cache"
        sudo /vagrant/scripts/update_classes.sh
      else
        sudo /usr/local/bin/puppet agent -t
      fi
    SHELL
  end

  # Agents
  if agents > 0
    (1..agents).each do |i|
      config.vm.define "agent#{i}" do |agent|
        agent.vm.box = box
        agent.vm.hostname = "agent#{i}.#{domain}"
        agent.vm.network "private_network", type: "dhcp"
        agent.vm.provider "virtualbox" do |vb|
          vb.memory = "512"
        end
        agent.vm.provision :hosts do |prov|
          prov.autoconfigure = true
        end
        agent.vm.provision "shell", inline: <<-SHELL
          # Stop iptables
          sudo service iptables stop
          sudo chkconfig iptables off
          # Install puppet
          /usr/local/bin/puppet --version 2&> /dev/null
          if [ $? -ne 0 ]; then
            curl -s -k https://master.#{domain}:8140/packages/current/install.bash | sudo bash
          else
            sudo /usr/local/bin/puppet agent -t
          fi
        SHELL
      end
    end
  end
end
