
# nmi_watchdog
# centos7.3 3.10.0-514.26.2.el7.x86_64

```
tee /etc/sysctl.d/kernel.watchdog_thresh.conf << 'EOF'
kernel.watchdog_thresh = 60
#kernel.unknown_nmi_panic = 0  # disable unknown nmi watchdog
#kernel.nmi_watchdog = 0       # disable nmi watchdog
EOF
```


```
[root@awx ~]# cat /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet nmi_watchdog=nopanic"
GRUB_DISABLE_RECOVERY="true"

[root@awx ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
[root@awx ~]# reboot

[root@awx ~]# cat /proc/cmdline
BOOT_IMAGE=/vmlinuz-3.10.0-514.26.2.el7.x86_64 root=UUID=7e4dd5ec-630f-4e99-b5cc-3cc7255c3337 ro rhgb quiet nmi_watchdog=nopanic
```
