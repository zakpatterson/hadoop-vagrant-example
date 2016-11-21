VAGRANT_API_VERSION="2"

MASTER_IP               = '192.168.1.201'
slaves = ['192.168.1.202', '192.168.1.203']


# Ensure role dependencies are in place
if [ "up", "provision" ].include?(ARGV.first) && !(File.directory?("ansible/roles/sbt"))
  unless system("ansible-galaxy install --force -r requirements.yml -p ansible/roles")
    $stderr.puts "\nERROR: Please install Ansible 1.4.2+ so that the ansible-galaxy binary"
    $stderr.puts "is available."
    exit(1)
  end
end

Vagrant.configure(VAGRANT_API_VERSION) do |config|

  #build master node. 
  config.vm.define "master" do |master|
    master.vm.provision "ansible" do |ansible|
      master.vm.hostname = "example.master"
      master.vm.network :public_network, ip: MASTER_IP    
      master.vm.box = "ubuntu/trusty64"

      master.vm.provider :virtualbox do |v|
        v.gui = false
        v.memory = 2048
        v.cpus = 2      
        
        v.customize [
          "modifyvm", :id,
          "--cpuexecutioncap", "75"
        ]
      end

      ansible.verbose = "vvv"    
      ansible.extra_vars = {
        "hadoop_master" => MASTER_IP,
        "hadoop_type_of_node" => "master"
      }

      ansible.playbook = "ansible/local.yml"
    end
  end

  #build slave nodes
  slaves.each_index do |machine_idx|
    config.vm.define "data#{machine_idx}" do |machine|
      machine.vm.provision "ansible" do |ansible|
        ansible.verbose = "vvv"    
        ansible.extra_vars = {
          "hadoop_master" => MASTER_IP,
          "hadoop_type_of_node" => "slave"
        }

        ansible.playbook = "ansible/local.yml"

        machine.vm.hostname = "example.data#{machine_idx}"
        machine.vm.network :public_network, ip: slaves[machine_idx]
        machine.vm.box = "ubuntu/trusty64"

        machine.vm.provider :virtualbox do |v|
          v.gui = false
          v.memory = 2048
          v.cpus = 2      
          
          v.customize [
            "modifyvm", :id,
            "--cpuexecutioncap", "75",
          ]
        end
      end
    end
  end
end
