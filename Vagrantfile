# Vagrant 2.3.4
# VirtualBox 6.1.40
# ubuntu 22.04 Jammy
$vm_box         = "ubuntu/jammy64"
$vm_box_url     = "https://s3.jcloud.sjtu.edu.cn/899a892efef34b1b944a19981040f55b-oss01/rsync/ubuntu-cloud-images/2675059070fdea94648b15eb4f302d8771fe0fac"

# kubernetes版本号
# 支持 v1.24.0 或更高版本，其他版本未经测试
$kube_version   = "1.28.0"

# 使用 VBoxManage list hostonlyifs 命令查看分配给虚拟机的网段
# ip地址从192.168.56.100的开始递增
$ip_range       = "192.168.58."
$ip_start       = 30
$master_ip      = $ip_range + "#{$ip_start}"

# master配置
$master_cpus    = 2
$master_memory  = 2048

# worker配置 
$worker_count   = 2
$worker_cpus    = 2
$worker_memory  = 2048

# k8s令牌
$k8s_token      = "123456.0123456789abcdef"
$net_iface      = "enp0s8"

# hostname
$master_host    = "k8s-master"
$worker_host    = "k8s-worker"

# pod使用的网段
$pod_subnet     = "172.10.0.0/16"

# 虚拟机配置
Vagrant.configure("2") do |config|

  # config.ssh.username = "vagrant"
  # config.ssh.password = "123456"

  # 通用配置
  config.vm.box = $vm_box
  config.vm.box_url = $vm_box_url

  # 通用初始化脚本
  config.vm.provision "shell" do |script|
    script.path = "bootstrap.sh"
    script.args = [ $kube_version ]
  end

  # 显卡控制器使用VMSVGA
  config.vm.provider :virtualbox do |vbox|
    vbox.customize ['modifyvm', :id, '--graphicscontroller', 'vmsvga']
  end

  # ===== 主节点配置 =====
  config.vm.define $master_host do |master|

    master.vm.hostname = $master_host
    master.vm.network "private_network", ip: $ip_range + "#{$ip_start}"

    # 规格配置
    master.vm.provider :virtualbox do |vbox|
      vbox.name    = $master_host
      vbox.cpus    = $master_cpus
      vbox.memory  = $master_memory
    end

    # kubeadmin 安装控制平面
    master.vm.provision "shell" do |script|
      script.path = "kubeadm-init.sh"
      script.args = [ $master_ip, $k8s_token, $net_iface, $pod_subnet, $kube_version ]
    end

  end
  # ===== 主节点配置 END =====

  # ===== worker节点配置 =====
  (1..$worker_count).each do |i|

    host_name = $worker_host + "#{i}"

    config.vm.define "#{host_name}" do |worker|
      worker.vm.hostname = "#{host_name}"
      worker.vm.network "private_network", ip: $ip_range + "#{$ip_start+i}"

      # 规格配置
      worker.vm.provider :virtualbox do |vbox|
        vbox.name    = "#{host_name}"
        vbox.cpus    = $worker_cpus
        vbox.memory  = $worker_memory
      end

      # 加入node节点
      worker.vm.provision "shell" do |script|
        script.inline = <<-SHELL
            kubeadm join $1:6443 --token $2 --discovery-token-unsafe-skip-ca-verification
          SHELL
        script.args = [ $master_ip, $k8s_token ]
      end
    end
  end
  # ===== worker节点配置 END =====

end