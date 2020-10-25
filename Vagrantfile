# 우분투 이미지
IMAGE_NAME = "bento/ubuntu-18.04"
MASTER_IP = "172.16.10.10"
NODE_IP = "172.16.10."
ANSIBLE_IP = "172.16.10.240"
# 워커 노드 수
N = 1

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    # 게스트 사양
    # 램 2GB
    # CPU 코어 2
    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end

    # 마스터 노드
    config.vm.define "master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: MASTER_IP, virtualbox__intnet: "kubernetes"
        master.vm.network :forwarded_port, guest: 22, host: 4444, id: 'ssh'
        master.vm.hostname = "master"
        master.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install sshpass
        SHELL
    end

    # 워커 노드
    (1..N).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: NODE_IP + "#{i + 10}", virtualbox__intnet: "kubernetes"
            node.vm.hostname = "node-#{i}"      
            node.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install sshpass
            SHELL
        end
    end

    # ansible 서버
    config.vm.define "ansible-server" do |node|
        node.vm.box = IMAGE_NAME
        node.vm.network "private_network", ip: ANSIBLE_IP, virtualbox__intnet: "kubernetes"
        node.vm.hostname = "ansible-server"           
        node.vm.provision "shell", inline: <<-SHELL
            sudo apt-get update
            sudo apt-get install software-properties-common
            sudo apt-add-repository --yes --update ppa:ansible/ansible
            sudo apt-get install ansible --yes
        SHELL
        # ansible 설정
        node.vm.provision "file", source: "./ansible_playbooks/setup-ansible.yml", destination: "setup.yml"
        node.vm.provision "shell", inline: "ansible-playbook setup.yml"
        # ssh 설정
        node.vm.provision "file", source: "./ansible_playbooks/configure_ssh.yml", destination: "configure_ssh.yml"
        node.vm.provision "shell", inline: "ansible-playbook configure_ssh.yml", privileged: false
        # 마스터노드 npm, nodejs설치
        node.vm.provision "file", source: "./ansible_playbooks/install_nodejs.yml", destination: "install_nodejs.yml"
        node.vm.provision "shell", inline: "ansible-playbook install_nodejs.yml", privileged: false
        # 마스터노드 쿠버네티스 설치 스크립트
        node.vm.provision "file", source: "./ansible_playbooks/kubernetes_masternode_install.yml", destination: "install_masternode.yml"
        node.vm.provision "shell", inline: "ansible-playbook install_masternode.yml", privileged: false
        # 워커노트 생성 및 쿠버네티스 설치
        node.vm.provision "file", source: "./ansible_playbooks/kubernetes_workernode_install.yml", destination: "install_workernode.yml"
        node.vm.provision "shell", inline: "ansible-playbook install_workernode.yml", privileged: false
    end
end