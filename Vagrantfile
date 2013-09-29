# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    # Box configuration
    config.vm.box = "precise64"
    config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    
    # Port Forwarding
    config.vm.network :forwarded_port, guest: 80, host: 5100
    config.vm.network :forwarded_port, guest: 5984, host: 5200

    # Network configuration
    config.vm.network :private_network, ip: "10.0.5.50"
    
    config.ssh.forward_agent = true

    # Enable NFS if your system supports it
    config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", :nfs => true

    # Virtual Box Custom configurations
    config.vm.provider :virtualbox do |vb|
        vb.customize [
            'modifyvm', :id,
            '--chipset', 'ich9',
            '--natdnsproxy1', 'on',
            '--natdnshostresolver1', 'on',
            '--memory', '1024'
        ]
    end

    # Provisioning the VM
    if Dir.glob("#{File.dirname(__FILE__)}/.vagrant/machines/default/*/id").empty?
        
        # Install nodejs
        pkg_cmd =   "echo 'Downloading and Installing Node.JS...';cd /tmp;wget -q http://nodejs.org/dist/v0.10.18/node-v0.10.18-linux-x64.tar.gz;"
        pkg_cmd <<  "tar -xvf node-v0.10.18-linux-x64.tar.gz; rm node-v0.10.18-linux-x64.tar.gz;"
        pkg_cmd <<  "sudo mv node-v0.10.18-linux-x64 /usr/local/node-v0.10.18;"
        pkg_cmd <<  "sudo ln -sf /usr/local/node-v0.10.18/bin/* /usr/local/bin/;"

        # Install couchdb
        pkg_cmd <<  "echo 'Downloading and Installing CouchDB...';sudo adduser --quiet --disabled-login --disabled-password --no-create-home --gecos \"\" couchdb;"
        pkg_cmd <<  "sudo apt-get install -q -y build-essential erlang-base-hipe erlang-dev erlang-manpages erlang-eunit erlang-nox libicu-dev libmozjs-dev libcurl4-openssl-dev pkg-config links redis-server;"
        pkg_cmd <<  "sudo apt-get install -q -y libmozjs185-dev libicu-dev libcurl4-gnutls-dev libtool vim;"
        pkg_cmd <<  "cd /tmp;wget -q http://mirrors.fe.up.pt/pub/apache/couchdb/source/1.4.0/apache-couchdb-1.4.0.tar.gz;tar -xvf apache-couchdb-1.4.0.tar.gz;rm apache-couchdb-1.4.0.tar.gz;"
        pkg_cmd <<  "cd apache-couchdb-1.4.0;./configure;make && make install;"
        pkg_cmd <<  "sudo ln -s /usr/local/etc/logrotate.d/couchdb /etc/logrotate.d/couchdb;sudo ln -s /usr/local/etc/init.d/couchdb  /etc/init.d;sudo update-rc.d couchdb defaults;"
        pkg_cmd <<  "sudo chown -R couchdb:couchdb /usr/local/var/log/couchdb;sudo chown -R couchdb:couchdb /usr/local/var/lib/couchdb;sudo chown -R couchdb:couchdb /usr/local/var/run/couchdb;"
        pkg_cmd <<  "sudo /bin/sed -i -e 's/'127.0.0.1'\\b/'0.0.0.0'/g' /usr/local/etc/couchdb/default.ini;sudo service couchdb restart;"
        
        # Add lxc-docker package
        pkg_cmd << "wget -q -O - https://get.docker.io/gpg | apt-key add -;" \
            "echo deb http://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list;" \
            "apt-get update -qq; apt-get install -q -y --force-yes lxc-docker; "
            
        # Add Ubuntu raring backported kernel
        pkg_cmd << "apt-get update -qq; apt-get install -q -y linux-image-generic-lts-raring; "
        
        # Add guest additions if local vbox VM. As virtualbox is the default provider,
        # it is assumed it won't be explicitly stated.
        if ENV["VAGRANT_DEFAULT_PROVIDER"].nil? && ARGV.none? { |arg| arg.downcase.start_with?("--provider") }
            pkg_cmd << "apt-get install -q -y linux-headers-generic-lts-raring dkms; " \
                "echo 'Downloading VBox Guest Additions...'; " \
                "wget -q http://dlc.sun.com.edgesuite.net/virtualbox/4.2.12/VBoxGuestAdditions_4.2.12.iso; "
            # Prepare the VM to add guest additions after reboot
            pkg_cmd << "echo -e 'mount -o loop,ro /home/vagrant/VBoxGuestAdditions_4.2.12.iso /mnt\n" \
                "echo yes | /mnt/VBoxLinuxAdditions.run\numount /mnt\n" \
                 "rm /root/guest_additions.sh; ' > /root/guest_additions.sh; " \
                "chmod 700 /root/guest_additions.sh; " \
                "sed -i -E 's#^exit 0#[ -x /root/guest_additions.sh ] \\&\\& /root/guest_additions.sh#' /etc/rc.local; " \
                "echo 'Installation of VBox Guest Additions is proceeding in the background.'; " \
                "echo '\"vagrant reload\" can be used in about 2 minutes to activate the new guest additions.'; "
        end
        
        # Activate new kernel
        pkg_cmd << "shutdown -r +1; "
        config.vm.provision :shell, :inline => pkg_cmd
    end
end