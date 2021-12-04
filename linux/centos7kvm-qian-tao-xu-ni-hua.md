# CentOS7-KVM 嵌套虚拟化

- 查看是否开启嵌套虚拟化

  ```bash
  cat /sys/module/kvm_intel/parameters/nested
  # 输出Y:表示开启，输出N:表示没有开启
  ```

- 配置 kvm 内核参数，开启嵌套虚拟化

  ```bash
  cat << EOF > /etc/modprobe.d/kvm-nested.conf
  options kvm-intel nested=1
  options kvm-intel enable_shadow_vmcs=1
  options kvm-intel enable_apicv=1
  options kvm-intel ept=1
  EOF
  ```

- 重新加载 kvm 内核

  > 注意：需要关闭全部虚拟机

  ```bash
  modprobe -r kvm_intel
  modprobe -a kvm_intel
  ```

- 验证

  修改 kvm 虚拟机 cpu 模式为`host-modle` 或`host-passthrough`

  在虚拟机中执行`egrep -o '(vmx|svm)' /proc/cpuinfo`查看是否开启

- 参考链接

  https://www.cnblogs.com/EasonJim/p/9752137.html
