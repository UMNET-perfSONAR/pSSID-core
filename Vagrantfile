
# perfSONAR dev environment
#


hosts = [
      
      [ "pi-point",           "10.0.0.3" ]
      #[ "elastic",            "10.0.0.2" ]
      #[ "testpoint",          "10.0.0.4" ]

      # This is built last so it can provision the others
      #[ "controller",           "10.0.0.10" ]
]

# make this work properly
# XXX automate UID
private_network_name = "pSSID-core-"
private_network_name << ENV['USER']


etc_hosts = hosts.map { |host, ip| "#{ip} #{host}" }.join("\n")

Vagrant.configure("2") do |config|

  # The default E1000 has a security vulerability.
  # this suppresses a Vagrant complaint
  config.vm.provider "virtualbox" do |vbox|
    vbox.default_nic_type = "82543GC"
  end

  hosts.each do |name, ip|
    config.vm.define name do |host|
      host.vm.provider "virtualbox" do |vb|
        vb.cpus = 2
        vb.memory = 4096
        # Don't need the guest extensions on this host.
        if Vagrant.has_plugin?("vagrant-vbguest")
          config.vbguest.auto_update = false
        end
      end

      if name == "pi-point"
          host.vm.box = "ubuntu/bionic64"
      else # if name
          host.vm.box = "centos/7"
      end # if name

      host.vm.hostname = name
      host.vm.network "private_network", ip: ip, virtualbox__intnet: private_network_name

      # Fill the hosts file

      host.vm.provision "#{name}-hosts", type: "shell", run: "always", inline: <<-SHELL

        set -e

        fgrep localhost /etc/hosts > /etc/hosts.build
        echo "#{etc_hosts}" >> /etc/hosts.build
        mv -f /etc/hosts.build /etc/hosts
	
      SHELL
      
      if name == "elastic"
        host.vm.network "forwarded_port", guest: 15672, host: "15672", host_ip: "198.111.224.158"
        host.vm.network "forwarded_port", guest: 9200, host: "9200", host_ip: "198.111.224.158"
        host.vm.network "forwarded_port", guest:5601, host: "5601", host_ip: "198.111.224.158"
        # ELK setup goes here
      end 
       
   end  # Config
  end  # hosts.each

  config.vm.provision "ansible" do |ansible|
    ansible.verbose = "v"
    ansible.groups = {
      "all:vars" => {
	"ansible_become" => "True",
	"ansible_become_user" => "root",
	"ansible_become_method" => "sudo"
      },
      "pSSID-testpoints" => ["pi-point"],
      #"pSSID-testpoints:vars" => {
	#"perfsonar_archive_auth_interfaces" => "{{ ansible_all_ipv4_addresses }}",
	#"perfsonar_archive_hosts" => "{{ groups['ps-archives'] }}"
      #},
    }
    #ansible.limit = "all",
    ansible.playbook = "pSSID.yml"
  end

end
