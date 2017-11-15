nodes = {
    'haproxy': { count: 1, ip_start: 11, box: 'ubuntu/xenial64'},
    'backend': { count: 3, ip_start: 21, box: 'ubuntu/xenial64'}  
}
 
Vagrant.configure("2") do |config|
  backend_servers = Array.new(nodes[:backend][:count]) do |i| 
    { name: "backend-0#{i+1}", ip: "33.33.33.#{nodes[:backend][:ip_start]+i}" }
  end

  nodes.each do |prefix, cfg|
    cfg[:count].times do |i|
      hostname = "%s-%02d" % [prefix, (i+1)]
 
      config.vm.define hostname do |box|
        box.vm.box = cfg[:box]
        box.vm.host_name = "#{hostname}.vagrant.internal"
        box.vm.network :private_network, 
                       ip: "33.33.33.#{cfg[:ip_start]+i}"

        # box.ssh.insert_key = false

        box.vm.provider 'virtualbox' do |vb|
          vb.customize [ 'modifyvm', :id,
                         '--name', hostname, 
                         '--memory', cfg[:memory] || 512
                       ]
        end

        box.vm.provision "shell", :inline => <<-SHELL
            apt-get install -y python
        SHELL

        box.vm.provision 'ansible' do |ansible|
          ansible.extra_vars = {
            backend_servers: backend_servers
          }
          ansible.playbook = "playbooks/#{prefix}.yml"
        end
      end
    end
  end
end