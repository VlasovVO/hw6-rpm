# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :rpm => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.101',
    :disks => {
        :sata1 => {
            :dfile => home + '/VirtualBox VMs/sata1.vdi',
            :size => 10240,
            :port => 1
        },
        :sata2 => {
            :dfile => home + '/VirtualBox VMs/sata2.vdi',
            :size => 2048, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => home + '/VirtualBox VMs/sata3.vdi',
            :size => 1024, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => home + '/VirtualBox VMs/sata4.vdi',
            :size => 1024,
            :port => 4
        }
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  
            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "256"]
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
            yum install -y gcc redhat-lsb-core wget rpmdevtools rpm-build createrepo yum-utils
            wget https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
            http://vault.centos.org/7.6.1810/os/Source/SPackages/apache-commons-daemon-1.0.13-7.el7.src.rpm
            adduser builder
            adduser mockbuild
            rpm -i nginx-1.14.1-1.el7_4.ngx.src.rpm
            rpm -i apache-commons-daemon-1.0.13-7.el7.src.rpm
            wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1a.tar.gz
            tar -xvf openssl-1.1.1a.tar.gz
            cp -Rf ~vagrant/openssl-1.1.1a ~root/openssl-1.1.1a
            yum-builddep -y ~root/rpmbuild/SPECS/nginx.spec
            yum-builddep -y ~root/rpmbuild/SPECS/apache-commons-daemon.spec
            rm ~root/rpmbuild/SPECS/nginx.spec
            wget https://gist.githubusercontent.com/lalbrekht/6c4a989758fccf903729fc55531d3a50/raw/8104e513dd9403a4d7b5f1393996b728f8733dd4/gistfile1.txt -O ~root/rpmbuild/SPECS/nginx.spec
            rpmbuild -bb ~root/rpmbuild/SPECS/nginx.spec
            rpmbuild -bb rpmbuild/SPECS/apache-commons-daemon.spec
            yum localinstall -y ~root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm
            systemctl start nginx
            mkdir -p /usr/share/nginx/html/repo
            cp ~root/rpmbuild/RPMS/x86_64/nginx-1.14.1-1.el7_4.ngx.x86_64.rpm /usr/share/nginx/html/repo/
            cp ~root/rpmbuild/RPMS/x86_64/apache-commons-daemon-1.0.13-7.el7.centos.x86_64.rpm /usr/share/nginx/html/repo/
            wget http://www.percona.com/downloads/percona-release/redhat/0.1-6/percona-release-0.1-6.noarch.rpm -O /usr/share/nginx/html/repo/percona-release-0.1-6.noarch.rpm
            createrepo /usr/share/nginx/html/repo/
            rm /etc/nginx/conf.d/default.conf 
            wget https://gist.githubusercontent.com/VlasovVO/f9f56a72cd25981097fd6fcb885f07fb/raw/7318d64e28c06a49b11773416b4597c33c08c3a5/default.conf -O /etc/nginx/conf.d/default.conf
            nginx -s reload
            echo -e "[otus]\nname=otus-linux\nbaseurl=http://localhost/repo\ngpgcheck=0\nenabled=1\n" >> /etc/yum.repos.d/otus.repo
            yum repolist enabled | grep otus
            yum install percona-release -y
            yum install apache-commons-daemon -y
          SHELL
  
        end
    
    end
  end
  
