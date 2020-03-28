# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',
	:disks => {
        }

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "1024"]
                  needsController = false
		  boxconfig[:disks].each do |dname, dconf|
			  unless File.exist?(dconf[:dfile])
				vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                needsController =  true
                          end

		  end
                  if needsController == true
                     vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                     boxconfig[:disks].each do |dname, dconf|
                         vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                     end
                  end
          end
 	  box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
              useradd user1 && useradd user2 && useradd user3
              echo "Otus2019" | sudo passwd --stdin user1 
              echo "Otus2019" | sudo passwd --stdin user2
              echo "Otus2019" | sudo passwd --stdin user3
              groupadd admin
              usermod -G admin vagrant
              sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config
              systemctl restart sshd.service
              sed -i '/pam_nologin.so/a\ account required pam_exec.so /usr/local/bin/day_login.sh' /etc/pam.d/sshd
              cat <<'EOF' >> /usr/local/bin/day_login.sh
#!/bin/bash

grp=$(groups $PAM_USER)
for pkg in $grp; do
    if [ $pkg = "admin" ]; then
        exit 0
    fi
done

for weekend in Sun Sat; do
    if [ $(date +%a) = "$weekend" ]; then
        exit 1
    fi
done

exit 0
EOF
              chmod +x /usr/local/bin/day_login.sh
  	  SHELL
      end
  end
end
