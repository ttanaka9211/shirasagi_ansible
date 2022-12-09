# coding: utf-8
# -*- mode: ruby -*-

# Vagrant1.1+はAPI_VERSION=2を使用する
VAGRANTFILE_API_VERSION = "2"

# ディレクトリ定義
conf_dir = File.dirname(File.expand_path(__FILE__))
provision_dir = File.join(conf_dir, 'provisioning')

# config.ymlの取得
require 'yaml'
vconfig = YAML::load_file(File.join(conf_dir, "config.yml"))

# config.ymlを読み込み仮想マシンを設定
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # ホスト名を設定
  config.vm.hostname = vconfig['vagrant_hostname']
  # 自動設定
  if vconfig['vagrant_ip'] == "0.0.0.0" && Vagrant.has_plugin?("vagrant-auto_network")
    config.vm.network 'private_network', ip: vconfig['vagrant_ip'], auto_network: true
  # IPアドレスが指定されている場合（vagrant_ip）
  else
    config.vm.network 'private_network', ip: vconfig['vagrant_ip'], auto_config: true
  end

  # ディレクトリの同期
  # provisioning/group_vars/all.ymlのユーザのUIDと合わせる
  # groupはvboxsfマウントのためwheelとしておく
  config.vm.synced_folder vconfig['vagrant_synced_folder']['local_path'], vconfig['vagrant_synced_folder']['destination'], owner: 1002, group: "wheel", mount_options: ['dmode=775,fmode=775']

  # ansible-playbookを使用してプロビジョニングを実施
  config.vm.box = vconfig['vagrant_box']
  config.vm.box_version = vconfig['vagrant_box_version']

  # provisioning
  extra_vars = {
    "hostname" => vconfig['vagrant_hostname'],
  }
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = File.join(provision_dir, "vagrant_playbook.yml")
    ansible.groups = {
      "all:vars" => extra_vars
    }
  end

  # VirtualBox用設定
  config.vm.provider "virtualbox" do |v|
    v.name = vconfig['vagrant_hostname']
    v.memory = vconfig['vagrant_memory']
    v.cpus = vconfig['vagrant_cpus']
    v.customize ["modifyvm", :id, "--natdnshostresolver1", "on", "--ioapic", "on", "--paravirtprovider", "kvm"]
  end
end
