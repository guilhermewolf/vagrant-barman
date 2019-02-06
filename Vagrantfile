# -*- mode: ruby -*-
# vi: set ft=ruby :

servers = [ 
    {
	:name => "postgresql",
	:type => "master",
	:box => "ubuntu/xenial64",
	:box_version => "20180831.0.0",
	:eth1 => "192.168.205.10",
	:mem => "1024",
	:cpu => "1"
    },
    {
	:name => "barman",
	:type => "node",
        :box => "ubuntu/xenial64",
        :box_version => "20180831.0.0",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    }
]
$configureMaster = <<-SCRIPT

    # Install the postgres key
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
    sudo apt-get update
    sudo apt-get -y install postgresql-10
    echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
    sudo sed -i -e '1ihost    postgres          postgres          192.168.205.11/32          trust\' /etc/postgresql/10/main/pg_hba.conf
    sudo cat <<EOF >> /etc/postgresql/10/main/postgresql.conf
listen_addresses = '192.168.205.10'
archive_mode = on
wal_level = 'replica'
archive_command = 'rsync -a %p barman@192.168.205.11:/var/lib/barman/pg/incoming/%f'
max_wal_senders = 2
max_replication_slots = 2
EOF
   sudo systemctl restart postgresql
   mkdir /var/lib/postgresql/.ssh/
   cp /vagrant/files/id* /var/lib/postgresql/.ssh/
   cp /vagrant/files/id_rsa.pub /var/lib/postgresql/.ssh/authorized_keys
   sudo ssh-keyscan -H 192.168.205.11 >> ~/.ssh/known_hosts
   sudo chown -R postgres. /var/lib/postgresql/.ssh
SCRIPT

$configureNode = <<-SCRIPT
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    sudo echo "deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main 10" >> /etc/apt/sources.list.d/pgdg.list
    sudo apt-get update
    sudo apt-get install barman rsync -y
    cat << EOF >/etc/barman.conf
[barman]
barman_home = /var/lib/barman
barman_user = barman
configuration_files_directory = /etc/barman.d
log_file = /var/log/barman/barman.log
compression = gzip
backup_method = rsync
reuse_backup = link
archiver = on
EOF
    cat << EOF > /etc/barman.d/pg.conf
[pg]
description = "PG Backup"
conninfo = host=192.168.205.10 user=postgres dbname=postgres
ssh_command = ssh postgres@192.168.205.10
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
wal_retention_policy = main
EOF
    sudo mkdir /var/lib/barman/.ssh 
    sudo cp /vagrant/files/id* /var/lib/barman/.ssh/
    sudo cp /vagrant/files/id_rsa.pub /var/lib/barman/.ssh/authorized_keys
    sudo chown -R barman. /var/lib/barman/.ssh
SCRIPT

Vagrant.configure("2") do |config|

    servers.each do |opts|
        config.vm.define opts[:name] do |config|

            config.vm.box = opts[:box]
            config.vm.box_version = opts[:box_version]
            config.vm.hostname = opts[:name]
            config.vm.network :private_network, ip: opts[:eth1]

            config.vm.provider "virtualbox" do |v|
                v.name = opts[:name]
                v.customize ["modifyvm", :id, "--groups", "/Ambiente Barman"]
                v.customize ["modifyvm", :id, "--memory", opts[:mem]]
                v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]

            end
            if opts[:type] == "master"
                config.vm.provision "shell", inline: $configureMaster
            else
                config.vm.provision "shell", inline: $configureNode
            end
        end

    end
end

