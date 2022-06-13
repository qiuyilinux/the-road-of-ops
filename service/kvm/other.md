## 1 修改虚拟机名称

```shell
cd /etc/libvirt/qemu
cp jenkins.xml jenkins-bak.xml
cd /data/kvm/
mv jenkins.qcow2 jenkins-bak.qcow2

virsh undefine jenkins
cd /etc/libvirt/qemu
vi jenkins-bak.xml
	...
	<name>jenkins-bak</name>
	...
	<source file='/data/kvm/jenkins-bak.qcow2'/>
	...
virsh define jenkins.xml
```

