---
description: 컨테이너 인프라 환경 구축을 위한 쿠버네티스/도커
---

# 2장. 테스트 환경 구성하기

## 도구

- VirtualBox
- Vagrant

## 사전 테스트

```shell
vagrant init
```

{% tabs %}
{% tab title="Vagrantfile" %}

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "sysnet4admin/CentOS-k8s"
  config.vm.box_version = "0.7.0"
end
```

{% endtab %}
{% endtabs %}

```shell
vagrant up # 가상머신 생성

vagrant ssh # 위에서 생성한 가상머신에 접속

$ uptime # 실행시간 확인하기
$ cat /etc/redhat-release # 운영 체제 종류 확인하기
$ exit # 가상머신 빠져나오기

vagrant destroy -f # 가상 머신 삭제 (+ 강제종료)
```

### ⚠️ mount: unknown filesystem type 'vboxsf'

```shell
# vagrant-vbguest 플러그인 설치
vagrant plugin install vagrant-vbguest
```

## 테스트 환경 구축

{% tabs %}
{% tab title="Vagrantfile" %}

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "m-k8s" do |cfg|
    cfg.vm.box = "sysnet4admin/CentOS-k8s"
    cfg.vm.box_version = "0.7.0"
    cfg.vm.provider "virtualbox" do |vb|
      vb.name = "m-k8s(github_SysNet4Admin)"
      vb.cpus = 2
      vb.memory = 2048
      vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
    end
    cfg.vm.host_name = "m-k8s"
    cfg.vm.network "private_network", ip: "192.168.1.10"
    cfg.vm.network "forwarded_port", guest: 22, host: 60010, auto_correct: true, id: "ssh"
    cfg.vm.synced_folder "../data", "/vagrant", disabled: true
    cfg.vm.provision "shell", path: "install_pkg.sh" # install_pkg.sh 실행
    cfg.vm.provision "file", source: "ping_2_nds.sh", destination: "ping_2_nds.sh" # ping_2_nds.sh 파일을 게스트의 홈 디렉터리(/home/vagrant)로 전달
    cfg.vm.provision "shell", path: "config.sh" # config.sh 실행
  end

  # Added Nodes
  (1..3).each do |i|
    config.vm.define "w#{i}-k8s" do |cfg|
      cfg.vm.box = "sysnet4admin/CentOS-k8s"
      cfg.vm.box_version = "0.7.0"
      cfg.vm.provider "virtualbox" do |vb|
        vb.name = "w#{i}-k8s(github_SysNet4Admin)"
        vb.cpus = 1
        vb.memory = 1024
        vb.customize ["modifyvm", :id, "--groups", "/k8s-SM(github_SysNet4Admin)"]
      end
      cfg.vm.host_name = "w#{i}-k8s"
      cfg.vm.network "private_network", ip: "192.168.1.10#{i}"
      cfg.vm.network "forwarded_port", guest: 22, host: "6010#{i}", auto_correct: true, id: "ssh"
      cfg.vm.synced_folder "../data", "/vagrant", disabled: true
      cfg.vm.provision "shell", path: "install_pkg.sh"
    end
  end
end
```

{% endtab %}
{% tab title="install_pkg.sh" %}

```shell
#! /usr/bin/env bash
yum install epel-release -y # EPEL 저장소 설치
yum install vim-enhanced -y # 코드 하이라이트 설치
```

{% endtab %}
{% endtab %}
{% tab title="ping_2_nds.sh" %}

```shell
ping 182.168.1.101 -c 3
ping 182.168.1.102 -c 3
ping 182.168.1.103 -c 3
```

{% endtab %}
{% endtab %}
{% tab title="config.sh" %}

```shell
chmod 744 ./ping_2_nds.sh
```

{% endtab %}
{% endtabs %}

```shell
vagrant up
vagrant ssh m-k8s

# m-k8s
$ ip addr show eth1 # IP가 192.168.1.10 로 제대로 설정되었는지 확인하기
$ yum repolist # EPEL 저장소가 구성되었는지 확인하기
$ vi .bashrc # 문법 하이라이트가 적용되었는지 확인하기

$ ./ping_2_nds.sh # 192.168.1.101~103 과 통신에 문제 없는지 확인하기

$ exit

vagrant destroy -f
```
