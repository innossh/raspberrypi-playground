# Raspberry Pi on QEMU

See https://github.com/dhruvvyas90/qemu-rpi-kernel/wiki .

## Memo

First launch the virtual machine by QEMU.

```console
$ wget https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/kernel-qemu-4.9.59-stretch
$ wget https://github.com/dhruvvyas90/qemu-rpi-kernel/raw/master/versatile-pb.dtb
$ wget https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2018-04-19/2018-04-18-raspbian-stretch-lite.zip
$ unzip 2018-04-18-raspbian-stretch-lite.zip

$ qemu-system-arm -kernel kernel-qemu-4.9.59-stretch -M versatilepb -cpu arm1176 -m 256 -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -dtb versatile-pb.dtb -hda 2018-04-18-raspbian-stretch-lite.img -net nic -net user,hostfwd=tcp::10022-:22
```

Put a file for ssh connection and change the password inside the virtual machine.

```console
$ sudo touch /boot/ssh
$ passwd
$ sudo poweroff
```

Then resize the image and launch the virtual machine again.

```
$ qemu-img resize 2018-04-18-raspbian-stretch-lite.img +4G
$ qemu-system-arm -kernel kernel-qemu-4.9.59-stretch -M versatilepb -cpu arm1176 -m 256 -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -dtb versatile-pb.dtb -hda 2018-04-18-raspbian-stretch-lite.img -net nic -net user,hostfwd=tcp::10022-:22
```

Set up the ssh connection.

```
$ ssh-keygen -t rsa -b 4096 -C "pi@yourname.local" -f ./pi_id_rsa
$ ssh-add ./pi_id_rsa
$ ssh-copy-id -i ./pi_id_rsa.pub -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p 10022 pi@127.0.0.1
$ ssh -F ssh_config qemu-pi echo ok
```

Finally you can run ansible playbooks.

```console
$ ansible qemu-pi -m ping
qemu-pi | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}

# Resize the file system
$ ansible-playbook pi-partition.yml

# Install Node.js
$ ansible-playbook pi-nodejs.yml
```

----

If you want to run the ansible playbook to the physical Raspberry Pi, modify `ssh_config` and `hosts.ini` as below.

```diff
--- a/hosts.ini
+++ b/hosts.ini
@@ -1,2 +1,3 @@
 [pi]
 qemu-pi
+real-pi
```

```diff
--- a/ssh_config
+++ b/ssh_config
@@ -9,6 +9,17 @@ Host qemu-pi
   IdentitiesOnly yes
   LogLevel FATAL

+Host real-pi
+  HostName raspberrypi.local
+  User pi
+  Port 22
+  UserKnownHostsFile /dev/null
+  StrictHostKeyChecking no
+  PasswordAuthentication no
+  IdentityFile ./pi_id_rsa
+  IdentitiesOnly yes
+  LogLevel FATAL
+
 Host *
   ControlMaster auto
   ControlPath ~/.ansible/cp/%h-%p-%r
```

Then run some playbook to the physical machine.

```console
$ ansible-playbook pi-nodejs.yml -l real-pi
```
