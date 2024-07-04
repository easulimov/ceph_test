Vagrant.configure(2) do |config|
  config.ssh.insert_key = false
  config.vagrant.plugins = "vagrant-libvirt"
  config.vm.provision "shell", inline: $script
  config.vm.provision "shell", inline: $ntpconfig

  config.vm.define "node01" do |node01|
    node01.vm.box = "generic/ubuntu2204"
    node01.vm.hostname = "node01"
    node01.vm.network "private_network", ip: "10.10.10.11" # Приватный IP-адрес для первой машины
    node01.vm.provider "libvirt" do |libvirt|
      libvirt.driver = "kvm"
      libvirt.memory = "2048"
      libvirt.cpus = "2"
      libvirt.management_network_name = "default"
      libvirt.management_network_address = "192.168.122.0/24"
      libvirt.management_network_mode = "nat"
      libvirt.storage :file, size: "10G", type: 'qcow2'
    end
  end

  config.vm.define "node02" do |node02|
    node02.vm.box = "generic/ubuntu2204"
    node02.vm.hostname = "node02"
    node02.vm.network "private_network", ip: "10.10.10.12" # Приватный IP-адрес для второй машины
    node02.vm.provider "libvirt" do |libvirt|
      libvirt.driver = "kvm"
      libvirt.memory = "2048"
      libvirt.cpus = "1"
      libvirt.management_network_name = "default"
      libvirt.management_network_address = "192.168.122.0/24"
      libvirt.management_network_mode = "nat"
      libvirt.storage :file, size: "10G", type: 'qcow2'
    end
  end

  config.vm.define "node03" do |node03|
    node03.vm.box = "generic/ubuntu2204"
    node03.vm.hostname = "node03"
    node03.vm.network "private_network", ip: "10.10.10.13" # Приватный IP-адрес для третьей машины
    node03.vm.provider "libvirt" do |libvirt|
      libvirt.driver = "kvm"
      libvirt.memory = "2048"
      libvirt.cpus = "2"
      libvirt.management_network_name = "default"
      libvirt.management_network_address = "192.168.122.0/24"
      libvirt.management_network_mode = "nat"
      libvirt.storage :file, size: "10G", type: 'qcow2''
    end
  end

  config.vm.define "client" do |client|
    client.vm.box = "generic/ubuntu2204"
    client.vm.hostname = "client"
    client.vm.network "private_network", ip: "10.10.10.14" # Приватный IP-адрес для третьей машины
    client.vm.provider "libvirt" do |libvirt|
      libvirt.driver = "kvm"
      libvirt.memory = "1024"
      libvirt.cpus = "1"
      libvirt.management_network_name = "default"
      libvirt.management_network_address = "192.168.122.0/24"
      libvirt.management_network_mode = "nat"
    end
  end
end


$script = <<-SCRIPT
sudo cat <<EOF > /etc/apt/sources.list
deb http://mirror.yandex.ru/ubuntu jammy main restricted universe multiverse
deb http://mirror.yandex.ru/ubuntu jammy-updates main restricted universe multiverse
deb http://mirror.yandex.ru/ubuntu jammy-backports main restricted universe multiverse
deb http://mirror.yandex.ru/ubuntu jammy-security main restricted universe multiverse
EOF

sudo apt update
sudo apt upgrade -y
sudo apt install -y mc wget curl zip tree vim apt-transport-https ca-certificates gnupg2 software-properties-common
export OS_VERSION=Ubuntu_20.04
export CRIO_VERSION=1.23
SCRIPT


$ntpconfig = <<-SCRIPT
sudo sed -i 's/^#NTP/NTP=ntp1.vniiftri.ru ntp2.vniiftri.ru ntp3.vniiftri.ru ntp4.vniiftri.ru/' /etc/systemd/timesyncd.conf
sudo systemctl restart systemd-timesyncd.service
sudo systemctl status systemd-timesyncd.service
SCRIPT