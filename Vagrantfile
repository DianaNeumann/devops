Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox"
  config.vm.box = "ubuntu/jammy64"

  # Количество требуемых машин
  N = 2

  # Порты для перенаправления
  ports = [{ guest: 5432, host: 5432 }, { guest: 5432, host: 5433 }]

  def create_server(config, hostname, ip, port, is_last)
    config.vm.define hostname do |server|
      server.vm.hostname = "ubuntu"
      server.vm.network "private_network", ip: ip
      server.vm.network "forwarded_port", guest: port[:guest], host: port[:host]
      server.vm.provision "shell", inline: <<-SHELL
        sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/#g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
      if is_last
        server.vm.provision "ansible" do |ansible|
          ansible.limit = "all"
          ansible.playbook = "./playbook.yml"
          ansible.inventory_path = "./inventories/hosts.yml"
        end
      end
    end
  end

  (0...N).each do |i|
    hostname = "server#{i+1}"
    ip = "192.168.56.4#{i}"
    port = ports[i]
    is_last = (i == N - 1)
    create_server(config, hostname, ip, port, is_last)
  end
end