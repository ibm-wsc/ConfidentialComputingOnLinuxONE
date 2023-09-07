# Start the GREP11 Server as a Secure Execution-enabled, HPVS 2.1.6 guest

## launch the HPVS 2.1.6 GREP11 server

You will start this section from your login session on the RHEL host, and will soon be instructed to switch to your Ubuntu KVM guest session.
But until then, start from this familiar window or tab:

<img src="../../../images/RHELHost.png" width="351" height="216" >

This fancy command figures out the last two characters of your assigned userid and is used in other commands in this section, so that the lab instructions will work for everybody:

   ``` bash
   suffix=$(temp=$(whoami) && echo ${temp: -2})
   ```

You aren't going to change anything here since it's already been defined for you by the instructors, but you can display the KVM guest definition of your HPVS 2.1.6 GREP11 Server:

   ``` bash
   sudo virsh dumpxml grep11se${suffix}
   ```

???- example "Definition of KVM guest for GREP11 Server"

      ```
      <domain type='kvm'>
        <name>grep11se02</name>
        <uuid>2315f8ea-a340-4506-abbf-ae04cf7ea868</uuid>
        <metadata>
          <libosinfo:libosinfo xmlns:libosinfo="http://libosinfo.org/xmlns/libvirt/domain/1.0">
            <libosinfo:os id="http://ubuntu.com/ubuntu/20.04"/>
          </libosinfo:libosinfo>
        </metadata>
        <memory unit='KiB'>3903488</memory>
        <currentMemory unit='KiB'>3903488</currentMemory>
        <vcpu placement='static'>2</vcpu>
        <os>
          <type arch='s390x' machine='s390-ccw-virtio-rhel8.2.0'>hvm</type>
          <boot dev='hd'/>
        </os>
        <cpu mode='host-model' check='partial'/>
        <clock offset='utc'/>
        <on_poweroff>destroy</on_poweroff>
        <on_reboot>restart</on_reboot>
        <on_crash>destroy</on_crash>
        <devices>
          <emulator>/usr/libexec/qemu-kvm</emulator>
          <disk type='file' device='disk'>
            <driver name='qemu' type='qcow2' iommu='on'/>
            <source file='/var/lib/libvirt/images/hpcr/student02/ibm-hyper-protect-container-runtime-23.6.2.qcow2'/>
            <backingStore/>
            <target dev='vda' bus='virtio'/>
            <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0000'/>
          </disk>
          <disk type='file' device='disk'>
            <driver name='qemu' type='raw' cache='none' io='native' iommu='on'/>
            <source file='/var/lib/libvirt/images/hpvslab/student02/ciiso.iso'/>
            <target dev='vdc' bus='virtio'/>
            <readonly/>
            <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0002'/>
          </disk>
          <controller type='pci' index='0' model='pci-root'/>
          <interface type='network'>
            <mac address='52:54:00:b1:e0:11'/>
            <source network='default'/>
            <model type='virtio'/>
            <driver name='vhost' iommu='on'/>
            <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0001'/>
          </interface>
          <console type='pty'>
            <target type='sclp' port='0'/>
          </console>
          <audio id='1' type='none'/>
          <memballoon model='none'/>
          <panic model='s390'/>
        </devices>
      </domain>
      ```

Start your GREP11 Server and attach to its console.  Watch the messages carefully.  You should not see any failures:

   ``` bash
   sudo virsh start grep11se${suffix} --console
   
   ```

???- example "This is what success looks like"

      ```
      Domain 'grep11se02' started
      Connected to domain 'grep11se02'
      Escape character is ^] (Ctrl + ])
      # HPL11 build:23.8.5 enabler:23.6.0
      # Tue Sep  5 21:07:01 UTC 2023
      # Machine Type/Plant/Serial: 8561/02/31A38
      # create new root partition...
      # encrypt root partition...
      # create root filesystem...
      # write OS to root disk...
      # decrypt user-data...
      2 token decrypted, 0 encrypted token ignored
      # run attestation...
      # set hostname...
      # finish root disk setup...
      # Tue Sep  5 21:07:29 UTC 2023
      # HPL11 build:23.8.5 enabler:23.6.0
      # HPL11099I: bootloader end
      hpcr-dnslookup[729]: HPL14000I: Network connectivity check completed successfully.
      hpcr-logging[871]: Configuring logging ...
      hpcr-logging[872]: Version [1.1.145]
      hpcr-logging[872]: Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      hpcr-logging[872]: HPL01010I: Logging has been setup successfully.
      hpcr-logging[871]: Logging has been configured
      hpcr-catch-success[1337]: VSI has started successfully.
      hpcr-catch-success[1337]: HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
      ```

You will have to enter the `Ctrl + ]` key-combination to break out of the console.

## verify that GREP11 server log messages are received by rsyslog

**The logging of the GREP11 server is going to the _rsyslog_ service that you configured on your Ubuntu guest, so switch to the terminal tab or window for your KVM standard guest.**

You should still be comfortably logged in on this tab or window:

<img src="../../../images/KVMGuest.png" width="351" height="217" />

The arguments to the _journalctl_ command here aren't the most elegant in the world, but, unless midnight passed since you started your GREP11 Server, you will be able to see messages in rsyslog from when you just started up your GREP11 Server:

   ``` bash
   journalctl --since today --no-pager
   ```

There are a lot of messages logged, a veritable trove of treasure for the curious.  Here is an example of what you should be able to see:

???- example "Log messages in rsyslog from starting the GREP11 Server"

      ```
      Sep 05 21:07:31 ubuntu2204 vpcnode[1777]: authentication probe
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Linux version 5.15.0-79-generic (buildd@bos02-s390x-016) (gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #86-Ubuntu SMP Mon Jul 10 16:19:54 UTC 2023 (Ubuntu 5.15.0-79.86-generic 5.15.111)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  setup: Linux is running under KVM in 64-bit mode
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  setup: Relocating AMODE31 section of size 0x00003000
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  setup: The maximum memory size is 3812MB
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  cpu: 2 configured CPUs, 0 standby CPUs
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Write protected kernel read-only data: 18692k
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Zone ranges:
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:    DMA      [mem 0x0000000000000000-0x000000007fffffff]
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:    Normal   [mem 0x0000000080000000-0x00000000ee3fffff]
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Movable zone start for each node
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Early memory node ranges
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:    node   0: [mem 0x0000000000000000-0x00000000ee3fffff]
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Initmem setup node 0 [mem 0x0000000000000000-0x00000000ee3fffff]
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  On node 0, zone Normal: 7168 pages in unavailable ranges
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  percpu: Embedded 32 pages/cpu s91904 r8192 d30976 u131072
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  pcpu-alloc: s91904 r8192 d30976 u131072 alloc=32*4096
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  pcpu-alloc: [0] 0 [0] 1 
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Built 1 zonelists, mobility grouping on.  Total pages: 960624
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Policy zone: Normal
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Kernel command line: panic=0 blacklist=virtio_rng swiotlb=262144 cloud-init=disabled console=ttyS0 printk.time=0 systemd.getty_auto=0 systemd.firstboot=0 module.sig_enforce=1 quiet loglevel=0 systemd.show_status=0
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Unknown kernel command line parameters "blacklist=virtio_rng cloud-init=disabled", will be passed to user space.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Dentry cache hash table entries: 524288 (order: 10, 4194304 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Inode-cache hash table entries: 262144 (order: 9, 2097152 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  mem auto-init: stack:off, heap alloc:on, heap free:off
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  software IO TLB: mapped [mem 0x000000005fffc000-0x000000007fffc000] (512MB)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Memory: 3266280K/3903488K available (11988K kernel code, 3212K rwdata, 6704K rodata, 5200K init, 1252K bss, 637208K reserved, 0K cma-reserved)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  SLUB: HWalign=256, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ftrace: allocating 34120 entries in 134 pages
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ftrace: allocated 134 pages with 3 groups
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  rcu: Hierarchical RCU implementation.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  rcu: #011RCU restricting CPUs from NR_CPUS=512 to nr_cpu_ids=2.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  #011Rude variant of Tasks RCU enabled.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  #011Tracing variant of Tasks RCU enabled.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=2
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NR_IRQS: 3, nr_irqs: 3, preallocated irqs: 3
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  clocksource: tod: mask: 0xffffffffffffffff max_cycles: 0x3b0a9be803b0a9, max_idle_ns: 1805497147909793 ns
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  random: crng init done
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Console: colour dummy device 80x25
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  printk: console [ttyS0] enabled
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  printk: console [ttysclp0] enabled
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Calibrating delay loop (skipped)... 24038.00 BogoMIPS preset
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  pid_max: default: 32768 minimum: 301
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  LSM: Security Framework initializing
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  landlock: Up and running.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Yama: becoming mindful.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  AppArmor: AppArmor initialized
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Mount-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Mountpoint-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  rcu: Hierarchical SRCU implementation.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  smp: Bringing up secondary CPUs ...
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  smp: Brought up 1 node, 2 CPUs
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  devtmpfs: initialized
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  futex hash table entries: 512 (order: 5, 131072 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_NETLINK/PF_ROUTE protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: initializing netlink subsys (disabled)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=2000 audit(1693948021.799:1): state=initialized audit_enabled=0 res=1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Spectre V2 mitigation: etokens
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  HugeTLB registered 1.00 MiB page size, pre-allocated 0 pages
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  iommu: Default domain type: Translated 
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  iommu: DMA domain TLB invalidation policy: strict mode 
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  SCSI subsystem initialized
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NetLabel: Initializing
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NetLabel:  domain hash size = 128
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NetLabel:  unlabeled traffic allowed by default
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  zpci: PCI is not supported because CPU facilities 69 or 71 are not available
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  VFS: Disk quotas dquot_6.6.0
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  AppArmor: AppArmor Filesystem Enabled
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_INET protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  IP idents hash table entries: 65536 (order: 7, 524288 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  tcp_listen_portaddr_hash hash table entries: 2048 (order: 3, 32768 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  TCP established hash table entries: 32768 (order: 6, 262144 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  TCP bind hash table entries: 32768 (order: 7, 524288 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  TCP: Hash tables configured (established 32768 bind 32768)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  MPTCP token hash table entries: 4096 (order: 4, 98304 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  UDP hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  UDP-Lite hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_UNIX/PF_LOCAL protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_XDP protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Trying to unpack rootfs image as initramfs...
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  kvm-s390: SIE is not available
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  hypfs: The hardware system does not support hypfs
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Initialise system trusted keyrings
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type blacklist registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  workingset: timestamp_bits=45 max_order=20 bucket_order=0
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  zbud: loaded
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  squashfs: version 4.0 (2009/01/31) Phillip Lougher
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  fuse: init (API version 7.34)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  integrity: Platform Keyring initialized
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type asymmetric registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Asymmetric key parser 'x509' registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Block layer SCSI generic (bsg) driver version 0.4 loaded (major 249)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  io scheduler mq-deadline registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  hvc_iucv: The z/VM IUCV HVC device driver cannot be used without z/VM
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  loop: module loaded
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  tun: Universal TUN/TAP device driver, 1.6
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  device-mapper: core: CONFIG_IMA_DISABLE_HTABLE is disabled. Duplicate IMA measurements will not be recorded in the IMA log.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  device-mapper: uevent: version 1.0.3
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  device-mapper: ioctl: 4.45.0-ioctl (2021-03-22) initialised: dm-devel@redhat.com
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  drop_monitor: Initializing network drop monitor service
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_INET6 protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Freeing initrd memory: 9828K
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Segment Routing with IPv6
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  In-situ OAM (IOAM) with IPv6
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  NET: Registered PF_PACKET protocol family
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type dns_resolver registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  cio: Channel measurement facility initialized using format extended (mode autodetected)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  sclp_sd: Store Data request failed (eq=2, di=3, response=0x40f0, flags=0x00, status=0, rc=-5)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ap: The hardware system does not support AP instructions
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  registered taskstats version 1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loading compiled-in X.509 certificates
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Build time autogenerated kernel key: 033cfe156234b615233dffd1cb0a66d4b6280b04'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Live Patch Signing: 14df34d1a87cf37625abec039ef2bf521249b969'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Kernel Module Signing: 88f752e560a1e0737e31163a466ad7b70a850c19'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  blacklist: Loading compiled-in revocation X.509 certificates
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing: 61482aa2830d0ab2ad5af10b7250da9033ddcef0'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2017): 242ade75ac4a15e50d50c84b0d45ff3eae707a03'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (ESM 2018): 365188c1d374d6b07c3c8f240f8ef722433d6a8b'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2019): c0746fd6c5da3ae827864651ad66ae47fe24b3e8'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v1): a8d54bbb3825cfb94fa13c9f8a594a195c107b8d'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v2): 4cf046892d6fd3c9a5b03f98d845f90851dc6a8c'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v3): 100437bb6de6e469b581e61cd66bce3ef4ed53af'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (Ubuntu Core 2019): c1d57b8f6b743f23ee41f4f7ee292f06eecadfb9'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  zswap: loaded using pool lzo/zbud
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type .fscrypt registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type fscrypt-provisioning registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Key type encrypted registered
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  AppArmor: AppArmor sha1 policy hashing enabled
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ima: No TPM chip found, activating TPM-bypass!
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loading compiled-in module X.509 certificates
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Loaded X.509 cert 'Build time autogenerated kernel key: 033cfe156234b615233dffd1cb0a66d4b6280b04'
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ima: Allocated hash algorithm: sha1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ima: No architecture policies found
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: Initialising EVM extended attributes:
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.selinux
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.SMACK64
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.SMACK64EXEC
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.SMACK64TRANSMUTE
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.SMACK64MMAP
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.apparmor
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.ima
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: security.capability
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  evm: HMAC attrs: 0x1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Freeing unused kernel image (initmem) memory: 5200K
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Write protected read-only-after-init data: 136k
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Checked W+X mappings: passed, no unexpected W+X pages found
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Run /init as init process
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:    with arguments:
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:      /init
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:    with environment:
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:      HOME=/
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:      TERM=linux
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:      blacklist=virtio_rng
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:      cloud-init=disabled
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  virtio_blk virtio0: [vda] 209715200 512-byte logical blocks (107 GB/100 GiB)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  GPT:Primary header thinks Alt. header is not at the end of the disk.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  GPT:8388607 != 209715199
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  GPT:Alternate GPT header not at the end of the disk.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  GPT:8388607 != 209715199
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  GPT: Use GNU Parted to correct GPT errors.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:   vda: vda1
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  virtio_blk virtio1: [vdb] 816 512-byte logical blocks (418 kB/408 KiB)
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ISO 9660 Extensions: Microsoft Joliet Level 3
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  ISO 9660 Extensions: RRIP_1991A
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  EXT4-fs (dm-0): re-mounted. Opts: (null). Quota mode: none.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  systemd 249.11-0ubuntu3.9 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT +GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY -P11KIT -QRENCODE +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified)
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Detected virtualization kvm.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Detected architecture s390x.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Hostname set to <student02-grep11server>.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Initializing machine ID from random generator.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Installed transient /etc/machine-id file.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  /lib/systemd/system/verify-disk-encryption-invoker.service:6: Special user nobody configured, this is not safe!
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  /lib/systemd/system/se-dnslookup.service:10: Special user nobody configured, this is not safe!
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  /lib/systemd/system/hpl-catch-success.service:13: Special user nobody configured, this is not safe!
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  /lib/systemd/system/hpl-catch-failed.service:10: Special user nobody configured, this is not safe!
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found ordering cycle on hpl-logging.target/verify-active
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on se-logging-sync.target/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on se-logging.service/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on basic.target/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on sockets.target/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on docker.socket/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Found dependency on se-registry-auth.service/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  se-registry-auth.service: Job sockets.target/start deleted to break ordering cycle starting with se-registry-auth.service/start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Queued start job for default target Multi-User System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Created slice Slice /system/modprobe.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Created slice Slice /system/systemd-fsck.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Created slice User and Session Slice.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Dispatch Password Requests to Console Directory Watch.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Forward Password Requests to Wall Directory Watch.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Set up automount Arbitrary Executable File Formats File System Automount Point.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Local Encrypted Volumes.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Path Units.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Remote File Systems.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Slice Units.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Swaps.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Local Verity Protected Volumes.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Syslog Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on fsck to fsckd communication Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on initctl Compatibility Named Pipe.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Journal Audit Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Journal Socket (/dev/log).
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Journal Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Network Service Netlink Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on udev Control Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on udev Kernel Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting Huge Pages File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting POSIX Message Queue File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting Kernel Debug File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting Kernel Trace File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Journal Service...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set the console keyboard layout...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Create List of Static Device Nodes...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module chromeos_pstore...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module configfs...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module drm...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module efi_pstore...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module fuse...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module pstore_blk...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module pstore_zone...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Module ramoops...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in OpenVSwitch configuration for cleanup being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting File System Check on Root Device...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load Kernel Modules...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Coldplug All udev Devices...
      Sep 05 21:07:32 ubuntu2204 systemd-journald[1777]:  Journal started
      Sep 05 21:07:32 ubuntu2204 systemd-journald[1777]:  Runtime Journal (/run/log/journal/a536ae21ee0d4480954246b1ce38e0dd) is 4.0M, max 32.0M, 28.0M free.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Journal Service.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted Huge Pages File System.
      Sep 05 21:07:32 ubuntu2204 systemd-fsck[1777]:  /dev/mapper/luks-c88abd09-279d-4108-b5df-6faba4dd318e: clean, 26377/6291456 files, 808668/25161728 blocks
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted POSIX Message Queue File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted Kernel Debug File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted Kernel Trace File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Create List of Static Device Nodes.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@chromeos_pstore.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module chromeos_pstore.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@efi_pstore.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module efi_pstore.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@fuse.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module fuse.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@pstore_blk.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module pstore_blk.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@pstore_zone.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module pstore_zone.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished File System Check on Root Device.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Modules.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting FUSE Control File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started File System Check Daemon to report status.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Remount Root and Kernel File Systems...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Apply Kernel Variables...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@configfs.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module configfs.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@drm.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module drm.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  modprobe@ramoops.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load Kernel Module ramoops.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting Kernel Configuration File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted FUSE Control File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted Kernel Configuration File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Coldplug All udev Devices.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  EXT4-fs (dm-0): re-mounted. Opts: errors=remount-ro. Quota mode: none.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Remount Root and Kernel File Systems.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Flush Journal to Persistent Storage...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in Platform Persistent Storage Archival being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load/Save Random Seed...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Create System Users...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Apply Kernel Variables.
      Sep 05 21:07:32 ubuntu2204 systemd-journald[1777]:  Time spent on flushing to /var/log/journal/a536ae21ee0d4480954246b1ce38e0dd is 2.005ms for 272 entries.
      Sep 05 21:07:32 ubuntu2204 systemd-journald[1777]:  System Journal (/var/log/journal/a536ae21ee0d4480954246b1ce38e0dd) is 8.0M, max 4.0G, 3.9G free.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Create System Users.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Create Static Device Nodes in /dev...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load/Save Random Seed.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in First Boot Complete being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Create Static Device Nodes in /dev.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Rule-based Manager for Device Events and Files...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set the console keyboard layout.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Preparation for Local File Systems.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Flush Journal to Persistent Storage.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Rule-based Manager for Device Events and Files.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Network Configuration...
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  VFIO - User Level meta-driver version: 0.3
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  lo: Link UP
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  lo: Gained carrier
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  Enumeration completed
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Network Configuration.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Wait for Network to be Configured...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Wait for Network to be Configured.
      Sep 05 21:07:32 ubuntu2204 systemd-udevd[1777]:  Using default interface naming scheme 'v249'.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  virtio_net virtio2 enc1: renamed from eth0
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  eth0: Interface name change detected, renamed to enc1.
      Sep 05 21:07:32 ubuntu2204 systemd-udevd[1777]:  Using default interface naming scheme 'v249'.
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  enc1: Link UP
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Found device /dev/disk/by-uuid/4d7e976d-b69c-48ec-9a8a-a47cd2e28e70.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting File System Check on /dev/disk/by-uuid/4d7e976d-b69c-48ec-9a8a-a47cd2e28e70...
      Sep 05 21:07:32 ubuntu2204 systemd-fsck[1777]:  /dev/vda1: clean, 13/262144 files, 140195/1048064 blocks
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished File System Check on /dev/disk/by-uuid/4d7e976d-b69c-48ec-9a8a-a47cd2e28e70.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting /boot...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted /boot.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Local File Systems.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Load AppArmor profiles...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set console font and keymap...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set Up Additional Binary Formats...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in Store a System Token in an EFI Variable being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Commit a transient machine-id on disk...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Create Volatile Files and Directories...
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Sep 05 21:07:32 ubuntu2204 apparmor.systemd[1777]:  Restarting AppArmor
      Sep 05 21:07:32 ubuntu2204 apparmor.systemd[1777]:  Reloading AppArmor profiles
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set console font and keymap.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  proc-sys-fs-binfmt_misc.automount: Got automount request for /proc/sys/fs/binfmt_misc, triggered by 702 (systemd-binfmt)
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounting Arbitrary Executable File Formats File System...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Create Volatile Files and Directories.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Network Name Resolution...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Network Time Synchronization...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Record System Boot/Shutdown in UTMP...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Mounted Arbitrary Executable File Formats File System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set Up Additional Binary Formats.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Record System Boot/Shutdown in UTMP.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Commit a transient machine-id on disk.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Network Time Synchronization.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target System Time Set.
      Sep 05 21:07:32 ubuntu2204 systemd-resolved[1777]:  Positive Trust Anchors:
      Sep 05 21:07:32 ubuntu2204 systemd-resolved[1777]:  . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
      Sep 05 21:07:32 ubuntu2204 systemd-resolved[1777]:  Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172.in-addr.arpa 20.172.in-addr.arpa 21.172.in-addr.arpa 22.172.in-addr.arpa 23.172.in-addr.arpa 24.172.in-addr.arpa 25.172.in-addr.arpa 26.172.in-addr.arpa 27.172.in-addr.arpa 28.172.in-addr.arpa 29.172.in-addr.arpa 30.172.in-addr.arpa 31.172.in-addr.arpa 168.192.in-addr.arpa d.f.ip6.arpa corp home internal intranet lan local private test
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=717 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 systemd-resolved[1777]:  Using system hostname 'student02-grep11server'.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Network Name Resolution.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Network.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Network is Online.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Host and Network Name Lookups.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.519:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=717 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=718 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=718 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 apparmor.systemd[1777]:  Skipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.549:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=718 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.549:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=718 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Load AppArmor profiles.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target System Initialization.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Daily apt download activities.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Daily apt upgrade and clean activities.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Daily dpkg database backup timer.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Periodic ext4 Online Metadata Check for All Filesystems.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Discard unused blocks once a week.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Daily rotation of log files.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Message of the Day.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Podman auto-update timer.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Daily Cleanup of Temporary Directories.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in Ubuntu Pro Timer for running repeated jobs being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Timer for calling verify disk encryption invoker service.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Basic System.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Timer Units.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on D-Bus System Message Bus Socket.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Podman API Socket.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.789:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.789:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.789:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948049.789:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=719 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting containerd container runtime...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started D-Bus System Message Bus.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Save initial kernel messages after boot.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Remove Stale Online ext4 Metadata Check Snapshots...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in getty on tty2-tty6 if dbus and logind are not available being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Login Prompts.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Dispatcher daemon for systemd-networkd...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Podman auto-update service...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Podman Start All Containers With Restart Policy Set To Always...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Podman API Service...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Logging Configuration...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting User Login Management...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Permit User Sessions...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in Ubuntu Pro reboot cmds being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Condition check resulted in Ubuntu Pro Background Auto Attach being skipped.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Podman API Service.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Permit User Sessions.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set console scheme...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set console scheme.
      Sep 05 21:07:32 ubuntu2204 dbus-daemon[1777]:  [system] AppArmor D-Bus mediation is enabled
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  e2scrub_reap.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Remove Stale Online ext4 Metadata Check Snapshots.
      Sep 05 21:07:32 ubuntu2204 systemd-logind[1777]:  New seat seat0.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started User Login Management.
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.948346972Z" level=info msg="starting containerd" revision= version=1.7.2
      Sep 05 21:07:32 ubuntu2204 networkd-dispatcher[1777]:  No valid path found for iwconfig
      Sep 05 21:07:32 ubuntu2204 networkd-dispatcher[1777]:  No valid path found for iw
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Dispatcher daemon for systemd-networkd.
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.986902685Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." type=io.containerd.snapshotter.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.992601773Z" level=info msg="skip loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." error="path /var/lib/containerd/io.containerd.snapshotter.v1.btrfs (ext4) must be a btrfs filesystem to be used with the btrfs snapshotter: skip plugin" type=io.containerd.snapshotter.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.992655035Z" level=info msg="loading plugin \"io.containerd.content.v1.content\"..." type=io.containerd.content.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.992737527Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.native\"..." type=io.containerd.snapshotter.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.992815968Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.overlayfs\"..." type=io.containerd.snapshotter.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.994445610Z" level=info msg="loading plugin \"io.containerd.metadata.v1.bolt\"..." type=io.containerd.metadata.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:29.994480649Z" level=info msg="metadata content store policy set" policy=shared
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:29Z" level=info msg="/usr/bin/podman filtering at log level info"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:29Z" level=info msg="/usr/bin/podman filtering at log level info"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004455490Z" level=info msg="loading plugin \"io.containerd.differ.v1.walking\"..." type=io.containerd.differ.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004488438Z" level=info msg="loading plugin \"io.containerd.event.v1.exchange\"..." type=io.containerd.event.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004503586Z" level=info msg="loading plugin \"io.containerd.gc.v1.scheduler\"..." type=io.containerd.gc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004540771Z" level=info msg="loading plugin \"io.containerd.lease.v1.manager\"..." type=io.containerd.lease.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004553332Z" level=info msg="loading plugin \"io.containerd.nri.v1.nri\"..." type=io.containerd.nri.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004591270Z" level=info msg="NRI interface is disabled by configuration."
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004600441Z" level=info msg="loading plugin \"io.containerd.runtime.v2.task\"..." type=io.containerd.runtime.v2
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004665859Z" level=info msg="loading plugin \"io.containerd.runtime.v2.shim\"..." type=io.containerd.runtime.v2
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004679908Z" level=info msg="loading plugin \"io.containerd.sandbox.store.v1.local\"..." type=io.containerd.sandbox.store.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004693273Z" level=info msg="loading plugin \"io.containerd.sandbox.controller.v1.local\"..." type=io.containerd.sandbox.controller.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004703656Z" level=info msg="loading plugin \"io.containerd.streaming.v1.manager\"..." type=io.containerd.streaming.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004716116Z" level=info msg="loading plugin \"io.containerd.service.v1.introspection-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004725918Z" level=info msg="loading plugin \"io.containerd.service.v1.containers-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004736897Z" level=info msg="loading plugin \"io.containerd.service.v1.content-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004751456Z" level=info msg="loading plugin \"io.containerd.service.v1.diff-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004761696Z" level=info msg="loading plugin \"io.containerd.service.v1.images-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004770993Z" level=info msg="loading plugin \"io.containerd.service.v1.namespaces-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004780547Z" level=info msg="loading plugin \"io.containerd.service.v1.snapshots-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004790646Z" level=info msg="loading plugin \"io.containerd.runtime.v1.linux\"..." type=io.containerd.runtime.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.004835598Z" level=info msg="loading plugin \"io.containerd.monitor.v1.cgroups\"..." type=io.containerd.monitor.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005053689Z" level=info msg="loading plugin \"io.containerd.service.v1.tasks-service\"..." type=io.containerd.service.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005079036Z" level=info msg="loading plugin \"io.containerd.grpc.v1.introspection\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005089501Z" level=info msg="loading plugin \"io.containerd.transfer.v1.local\"..." type=io.containerd.transfer.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005107004Z" level=info msg="loading plugin \"io.containerd.internal.v1.restart\"..." type=io.containerd.internal.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005146580Z" level=info msg="loading plugin \"io.containerd.grpc.v1.containers\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005162499Z" level=info msg="loading plugin \"io.containerd.grpc.v1.content\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005176845Z" level=info msg="loading plugin \"io.containerd.grpc.v1.diff\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005220181Z" level=info msg="loading plugin \"io.containerd.grpc.v1.events\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005229711Z" level=info msg="loading plugin \"io.containerd.grpc.v1.healthcheck\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005239184Z" level=info msg="loading plugin \"io.containerd.grpc.v1.images\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005250961Z" level=info msg="loading plugin \"io.containerd.grpc.v1.leases\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005259808Z" level=info msg="loading plugin \"io.containerd.grpc.v1.namespaces\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005270364Z" level=info msg="loading plugin \"io.containerd.internal.v1.opt\"..." type=io.containerd.internal.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005600410Z" level=info msg="loading plugin \"io.containerd.grpc.v1.sandbox-controllers\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005619593Z" level=info msg="loading plugin \"io.containerd.grpc.v1.sandboxes\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005629434Z" level=info msg="loading plugin \"io.containerd.grpc.v1.snapshots\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005640996Z" level=info msg="loading plugin \"io.containerd.grpc.v1.streaming\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005650346Z" level=info msg="loading plugin \"io.containerd.grpc.v1.tasks\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005661167Z" level=info msg="loading plugin \"io.containerd.grpc.v1.transfer\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005673456Z" level=info msg="loading plugin \"io.containerd.grpc.v1.version\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005684487Z" level=info msg="loading plugin \"io.containerd.grpc.v1.cri\"..." type=io.containerd.grpc.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005783067Z" level=info msg="Start cri plugin with config {PluginConfig:{ContainerdConfig:{Snapshotter:overlayfs DefaultRuntimeName:runc DefaultRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false PrivilegedWithoutHostDevicesAllDevicesAllowed:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0 Snapshotter: SandboxMode:} UntrustedWorkloadRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false PrivilegedWithoutHostDevicesAllDevicesAllowed:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0 Snapshotter: SandboxMode:} Runtimes:map[runc:{Type:io.containerd.runc.v2 Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[BinaryName: CriuImagePath: CriuPath: CriuWorkPath: IoGid:0 IoUid:0 NoNewKeyring:false NoPivotRoot:false Root: ShimCgroup: SystemdCgroup:false] PrivilegedWithoutHostDevices:false PrivilegedWithoutHostDevicesAllDevicesAllowed:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0 Snapshotter: SandboxMode:podsandbox}] NoPivot:false DisableSnapshotAnnotations:true DiscardUnpackedLayers:false IgnoreBlockIONotEnabledErrors:false IgnoreRdtNotEnabledErrors:false} CniConfig:{NetworkPluginBinDir:/opt/cni/bin NetworkPluginConfDir:/etc/cni/net.d NetworkPluginMaxConfNum:1 NetworkPluginSetupSerially:false NetworkPluginConfTemplate: IPPreference:} Registry:{ConfigPath: Mirrors:map[] Configs:map[] Auths:map[] Headers:map[]} ImageDecryption:{KeyModel:node} DisableTCPService:true StreamServerAddress:127.0.0.1 StreamServerPort:0 StreamIdleTimeout:4h0m0s EnableSelinux:false SelinuxCategoryRange:1024 SandboxImage:registry.k8s.io/pause:3.8 StatsCollectPeriod:10 SystemdCgroup:false EnableTLSStreaming:false X509KeyPairStreaming:{TLSCertFile: TLSKeyFile:} MaxContainerLogLineSize:16384 DisableCgroup:false DisableApparmor:false RestrictOOMScoreAdj:false MaxConcurrentDownloads:3 DisableProcMount:false UnsetSeccompProfile: TolerateMissingHugetlbController:true DisableHugetlbController:true DeviceOwnershipFromSecurityContext:false IgnoreImageDefinedVolumes:false NetNSMountsUnderStateDir:false EnableUnprivilegedPorts:false EnableUnprivilegedICMP:false EnableCDI:false CDISpecDirs:[/etc/cdi /var/run/cdi] ImagePullProgressTimeout:1m0s DrainExecSyncIOTimeout:0s} ContainerdRootDir:/var/lib/containerd ContainerdEndpoint:/run/containerd/containerd.sock RootDir:/var/lib/containerd/io.containerd.grpc.v1.cri StateDir:/run/containerd/io.containerd.grpc.v1.cri}"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005833821Z" level=info msg="Connect containerd service"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005852221Z" level=info msg="using legacy CRI server"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005856641Z" level=info msg="using experimental NRI integration - disable nri plugin to prevent this"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.005886955Z" level=info msg="Get image filesystem path \"/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs\""
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006370465Z" level=info msg="loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." type=io.containerd.tracing.processor.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006395817Z" level=info msg="Start subscribing containerd event"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006425898Z" level=info msg="Start recovering state"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006466590Z" level=info msg="Start event monitor"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006476779Z" level=info msg="Start snapshots syncer"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006484085Z" level=info msg="Start cni network conf syncer for default"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006490388Z" level=info msg="Start streaming server"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006645556Z" level=info msg="skip loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." error="no OpenTelemetry endpoint: skip plugin" type=io.containerd.tracing.processor.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006655987Z" level=info msg="loading plugin \"io.containerd.internal.v1.tracing\"..." type=io.containerd.internal.v1
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006665801Z" level=info msg="skipping tracing processor initialization (no tracing plugin)" error="no OpenTelemetry endpoint: skip plugin"
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006790238Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.006806339Z" level=info msg=serving... address=/run/containerd/containerd.sock
      Sep 05 21:07:32 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:30.010390952Z" level=info msg="containerd successfully booted in 0.064417s"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started containerd container runtime.
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="[graphdriver] using prior storage driver: overlay"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="[graphdriver] using prior storage driver: overlay"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  2023-09-05 21:07:30.089577891 +0000 UTC m=+0.233357472 system refresh
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="Setting parallel job count to 7"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="Setting parallel job count to 7"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="using systemd socket activation to determine API endpoint"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="using API endpoint: ''"
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  time="2023-09-05T21:07:30Z" level=info msg="API service listening on \"/run/podman/podman.sock\""
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  etc-machine\x2did.mount: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Podman Start All Containers With Restart Policy Set To Always.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  podman-auto-update.service: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Podman auto-update service.
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  enc1: Gained carrier
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  IPv6: ADDRCONF(NETDEV_CHANGE): enc1: link becomes ready
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  enc1: DHCPv4 address 172.16.0.62/24 via 172.16.0.1
      Sep 05 21:07:32 ubuntu2204 dbus-daemon[1777]:  [system] Activating via systemd: service name='org.freedesktop.hostname1' unit='dbus-org.freedesktop.hostname1.service' requested by ':1.3' (uid=100 pid=647 comm="/lib/systemd/systemd-networkd " label="unconfined")
      Sep 05 21:07:32 ubuntu2204 systemd-timesyncd[1777]:  Network configuration changed, trying to establish connection.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Hostname Service...
      Sep 05 21:07:32 ubuntu2204 systemd-timesyncd[1777]:  Initial synchronization to time server 185.125.190.58:123 (ntp.ubuntu.com).
      Sep 05 21:07:32 ubuntu2204 dbus-daemon[1777]:  [system] Successfully activated service 'org.freedesktop.hostname1'
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Hostname Service.
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  Could not set hostname: Access denied
      Sep 05 21:07:32 ubuntu2204 hpcr-dnslookup[1777]:  HPL14000I: Network connectivity check completed successfully.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Logging Configuration.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Early Initialization.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Logging to remote monitoring server is initiated..
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Logging Configuration...
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  Configuring logging ...
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  Version [1.1.145]
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  ValidateContractE ...
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  config written: /etc/rsyslog.d/22-logging.conf
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  HPL01010I: Logging has been setup successfully.
      Sep 05 21:07:32 ubuntu2204 hpcr-logging[1777]:  Logging has been configured
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Logging Configuration.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting System Logging Service...
      Sep 05 21:07:32 ubuntu2204 rsyslogd[1777]:  rsyslogd's groupid changed to 111
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started System Logging Service.
      Sep 05 21:07:32 ubuntu2204 rsyslogd[1777]:  rsyslogd's userid changed to 104
      Sep 05 21:07:32 ubuntu2204 rsyslogd[1777]:  [origin software="rsyslogd" swVersion="8.2112.0" x-pid="876" x-info="https://www.rsyslog.com"] start
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Synchronizes the Logging Target.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Logging to remote log server is initiated..
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that does validation of contract...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting HPCR Registry Authentication...
      Sep 05 21:07:32 ubuntu2204 rsyslogd[1777]:  imjournal: No statefile exists, /var/spool/rsyslog/journal_state will be created (ignore if this is first run): No such file or directory [v8.2112.0 try https://www.rsyslog.com/e/2040 ]
      Sep 05 21:07:32 ubuntu2204 hpcr-registry-auth[1777]:  Starting Registry Authentication ...
      Sep 05 21:07:32 ubuntu2204 hpcr-contract[1777]:  Welcome to SE Contract Validator
      Sep 05 21:07:32 ubuntu2204 hpcr-contract[1777]:  Contract file passed is:  /var/hyperprotect/user-data.decrypted
      Sep 05 21:07:32 ubuntu2204 hpcr-registry-auth[1777]:  Version [1.0.70]
      Sep 05 21:07:32 ubuntu2204 hpcr-registry-auth[1777]:  Writing auth config: /root/.docker/config.json
      Sep 05 21:07:32 ubuntu2204 hpcr-registry-auth[1777]:  Registry Authentication started
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished HPCR Registry Authentication.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Docker Socket for the API...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Listening on Docker Socket for the API.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Docker Application Container Engine...
      Sep 05 21:07:32 ubuntu2204 rsyslogd[1777]:  imjournal: journal files changed, reloading...  [v8.2112.0 try https://www.rsyslog.com/e/0 ]
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.120373172Z" level=info msg="Starting up"
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.121621507Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so using resolv.conf: /run/systemd/resolve/resolv.conf"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 hpcr-contract[1777]:  Contract file is valid.
      Sep 05 21:07:32 ubuntu2204 hpcr-contract[1777]:  Extracting workload from /var/hyperprotect/user-data.decrypted to /var/hyperprotect/workload-data.decrypted
      Sep 05 21:07:32 ubuntu2204 hpcr-contract[1777]:  Extraction completed
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Service that does validation of contract.
      Sep 05 21:07:32 ubuntu2204 audit[1777]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=901 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.212965202Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.213035643Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.213072082Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.213099168Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.213993545Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.214032153Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.214063511Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.214087732Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  audit: type=1400 audit(1693948051.199:9): apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=901 comm="apparmor_parser"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that does signature validation of Env Workload of contract...
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Welcome to SE ENV Workload Signature Validator
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Decrypted Contract file passed is:  /var/hyperprotect/workload-data.decrypted
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Encrypted Contract file passed is:  /var/hyperprotect/cidata/user-data
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Check Dependency params Public key and EnvWorkload signature
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Access Public key and EnvWorkload signature
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Create combined EnvWorkload contract content
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Verify signing key, signature and combined EnvWorkload contract
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Verified OK
      Sep 05 21:07:32 ubuntu2204 hpcr-signature[1777]:  Successfully verified contract with signature and signing key
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Service that does signature validation of Env Workload of contract.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Contract is unpacked and ready for consumption..
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that waits until the user devices are ready...
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set podman image policy...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  Waiting for devices ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  Version [1.0.112]
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  WaitForDevices input=[/var/hyperprotect/user-data.decrypted], timeout=[2023-09-05 21:22:31.298142241 +0000 UTC m=+900.029451094]
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  ParseContract ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  ValidateContract ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  MergeVolumes ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-standby[1777]:  Waiting for devices is completed
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Service that waits until the user devices are ready.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that mounts the data volumes after they are ready...
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.308065659Z" level=info msg="Loading containers: start."
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  Mounting volumes ...
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  bridge: filtering via arp/ip/ip6tables is no longer available by default. Update your scripts to load br_netfilter if you need this.
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Bridge firewalling registered
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  Version [1.0.112]
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  MountVolumes input=[/var/hyperprotect/user-data.decrypted]
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  ParseContract ...
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Getting image source signatures
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Copying blob sha256:66d62867ae2452322f4769f943913be00b22e73039d1902e8f785b9f49838193
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  ValidateContract ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  MergeVolumes ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  Mounting volumes ...
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  Volume config ..
      Sep 05 21:07:32 ubuntu2204 hpcr-disk-mount[1777]:  HPL07003I: Mounting volumes done
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Service that mounts the data volumes after they are ready.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Reached target Data volumes are mounted ready to be used..
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Service that verifies all disks are encrypted and logs output to systemd journal.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Service that periodically logs entry to trigger verify disk encryption service.
      Sep 05 21:07:32 ubuntu2204 verify-disk-encryption[1777]:  Verify disk encryption started...
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Copying config sha256:9eca761232387055827db0a9f2232f2635bc8c6d5f23ecfb39d34bb4ab0dca09
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Writing manifest to image destination
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Storing signatures
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.476373977Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
      Sep 05 21:07:32 ubuntu2204 kernel[1777]:  Initializing XFRM netlink socket
      Sep 05 21:07:32 ubuntu2204 networkd-dispatcher[1777]:  WARNING:Unknown index 3 seen, reloading interface list
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Loaded image(s): k8s.gcr.io/pause:3.5
      Sep 05 21:07:32 ubuntu2204 podman[1777]:  2023-09-05 21:07:31.326771212 +0000 UTC m=+0.070314326 image loadfromarchive  /usr/local/se-image-play/pause.tar
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-image-play
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  docker0: Link UP
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  Version [1.1.112]
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:  pam_unix(sudo:session): session closed for user nobody
      Sep 05 21:07:32 ubuntu2204 hpcr-image-play[1777]:  tar: etc/containers/policy.json: time stamp 2023-09-05 21:07:32 is 0.466076487 s in the future
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set podman image policy.
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.539054768Z" level=info msg="Loading containers: done."
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that creates a set of containers...
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-container-play
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Sep 05 21:07:32 ubuntu2204 hpcr-container-play[1777]:  Version [1.1.116]
      Sep 05 21:07:32 ubuntu2204 sudo[1777]:  pam_unix(sudo:session): session closed for user nobody
      Sep 05 21:07:32 ubuntu2204 hpcr-container-play[1777]:  HPL15004I: The pod started successfully.
      Sep 05 21:07:32 ubuntu2204 hpcr-container-play[1777]:  HPL15006I: No pod definitions found.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Service that creates a set of containers.
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.612717146Z" level=info msg="Docker daemon" commit="20.10.25-0ubuntu1~22.04.2" graphdriver(s)=overlay2 version=20.10.25
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.612770737Z" level=info msg="Daemon has completed initialization"
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.643705432Z" level=info msg="API listen on /run/docker.sock"
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Started Docker Application Container Engine.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Set docker image policy...
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Starting image service...
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Extracting image contract
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Successfully extracted Image contract
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Extracting container contract
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Checking for image with digest
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  No image for DCT verification
      Sep 05 21:07:32 ubuntu2204 hpcr-image[1777]:  Image service completed successfully
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Finished Set docker image policy.
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  Starting Service that creates a set of containers...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Starting container service...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Validating contract...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Compose folder /data1/compose created
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Compose folder: /data1/compose
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Validation completed
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Parsing contract...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Parsing of the Contract File completed successfully
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Extracting compose...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Extracting done...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Extracting the ENV Contents...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Writing new env file /data1/compose/.env ...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Reading existing env file /data1/compose/.env ...
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Extracting of environment contents done
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Check if docker is ready
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  docker-compose.yml file is present in the directory
      Sep 05 21:07:32 ubuntu2204 hpcr-container[1777]:  Starting workload containers...
      Sep 05 21:07:32 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:31.912808357Z" level=warning msg="reference for unknown type: " digest="sha256:a864174faadc39650e61ca45d8a3ceb01ea88602cfe6f4bd4e35c48e60556900" remote="quay.io/gmoney23/grep11server@sha256:a864174faadc39650e61ca45d8a3ceb01ea88602cfe6f4bd4e35c48e60556900"
      Sep 05 21:07:32 ubuntu2204 systemd-networkd[1777]:  enc1: Gained IPv6LL
      Sep 05 21:07:32 ubuntu2204 systemd[1777]:  var-lib-docker-overlay2-opaque\x2dbug\x2dcheck1111129260-merged.mount: Deactivated successfully.
      Sep 05 21:07:34 ubuntu2204 networkd-dispatcher[1777]:  WARNING:Unknown index 4 seen, reloading interface list
      Sep 05 21:07:34 ubuntu2204 systemd-networkd[1777]:  br-35f22f910f6f: Link UP
      Sep 05 21:07:34 ubuntu2204 systemd[1777]:  var-lib-docker-overlay2-b0df35c73664d2a1dae1b726e9b3a17b4dceb966e3dc15a7c9362031c9d9b10e\x2dinit-merged.mount: Deactivated successfully.
      Sep 05 21:07:34 ubuntu2204 systemd[1777]:  dmesg.service: Deactivated successfully.
      Sep 05 21:07:35 ubuntu2204 systemd-udevd[1777]:  Using default interface naming scheme 'v249'.
      Sep 05 21:07:35 ubuntu2204 networkd-dispatcher[1777]:  WARNING:Unknown index 5 seen, reloading interface list
      Sep 05 21:07:35 ubuntu2204 systemd-networkd[1777]:  veth09c9325: Link UP
      Sep 05 21:07:35 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:34.926762864Z" level=info msg="No non-localhost DNS nameservers are left in resolv.conf. Using default external servers: [nameserver 8.8.8.8 nameserver 8.8.4.4]"
      Sep 05 21:07:35 ubuntu2204 dockerd[1777]:  time="2023-09-05T21:07:34.926902031Z" level=info msg="IPv6 enabled; Adding default IPv6 external servers: [nameserver 2001:4860:4860::8888 nameserver 2001:4860:4860::8844]"
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered blocking state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered disabled state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  device veth09c9325 entered promiscuous mode
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered blocking state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered forwarding state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered disabled state
      Sep 05 21:07:35 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:34.985760220Z" level=info msg="loading plugin \"io.containerd.internal.v1.shutdown\"..." runtime=io.containerd.runc.v2 type=io.containerd.internal.v1
      Sep 05 21:07:35 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:34.985814175Z" level=info msg="loading plugin \"io.containerd.ttrpc.v1.pause\"..." runtime=io.containerd.runc.v2 type=io.containerd.ttrpc.v1
      Sep 05 21:07:35 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:34.986027394Z" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runtime=io.containerd.runc.v2 type=io.containerd.event.v1
      Sep 05 21:07:35 ubuntu2204 containerd[1777]:  time="2023-09-05T21:07:34.986046098Z" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io.containerd.runc.v2 type=io.containerd.ttrpc.v1
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Started libcontainer container 5afe4af5b8a97851c54345117fabe0a3999e1761d03e1d7cd3f5c581f2a4d6b2.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  podman.service: Deactivated successfully.
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  eth0: renamed from vethe318d71
      Sep 05 21:07:35 ubuntu2204 systemd-networkd[1777]:  veth09c9325: Gained carrier
      Sep 05 21:07:35 ubuntu2204 systemd-networkd[1777]:  br-35f22f910f6f: Gained carrier
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  IPv6: ADDRCONF(NETDEV_CHANGE): veth09c9325: link becomes ready
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered blocking state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  br-35f22f910f6f: port 1(veth09c9325) entered forwarding state
      Sep 05 21:07:35 ubuntu2204 kernel[1777]:  IPv6: ADDRCONF(NETDEV_CHANGE): br-35f22f910f6f: link becomes ready
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Docker Compose Logs:
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:   student02-ep11server Pulling
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Pulling fs layer
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Waiting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Downloading [==================================>                ]     613B/890B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Downloading [==================================================>]     890B/890B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Extracting [==================================================>]     890B/890B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Extracting [==================================================>]     890B/890B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  620e494ced91 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Downloading [>                                                  ]  1.369kB/113.9kB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Downloading [==================================================>]  113.9kB/113.9kB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Extracting [==============>                                    ]  32.77kB/113.9kB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Extracting [==================================================>]  113.9kB/113.9kB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Extracting [==================================================>]  113.9kB/113.9kB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6c8a4d0d91d5 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Downloading [>                                                  ]  188.3kB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Downloading [====>                                              ]  1.554MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Downloading [=======>                                           ]   2.73MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Downloading [==================================================>]     167B/167B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Downloading [>                                                  ]  73.12kB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Downloading [=============================>                     ]  10.92MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Downloading [==========>                                        ]  1.521MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Downloading [============================================>      ]  16.79MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Downloading [=================>                                 ]   2.48MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Downloading [==================================================>]     174B/174B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Extracting [>                                                  ]  196.6kB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Extracting [========>                                          ]  3.146MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Downloading [================================>                  ]  4.602MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Extracting [===============>                                   ]  5.702MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Downloading [==================================================>]     156B/156B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Extracting [======================>                            ]  8.258MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Downloading [==================================================>]     139B/139B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Extracting [==================================================>]  18.69MB/18.69MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  870d2d701868 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Extracting [>                                                  ]   98.3kB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Downloading [==================================================>]     142B/142B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Extracting [================>                                  ]  2.359MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Downloading [==================================================>]     145B/145B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Verifying Checksum
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Download complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Extracting [=========================================>         ]    5.8MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Extracting [==================================================>]  6.994MB/6.994MB
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  17cad8585d31 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Extracting [==================================================>]     167B/167B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Extracting [==================================================>]     167B/167B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  957151557b52 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Extracting [==================================================>]     174B/174B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Extracting [==================================================>]     174B/174B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  356d6d7e116b Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Extracting [==================================================>]     156B/156B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Extracting [==================================================>]     156B/156B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  e90fb9a2f971 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Extracting [==================================================>]     139B/139B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Extracting [==================================================>]     139B/139B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  6792527a66d0 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Extracting [==================================================>]     145B/145B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Extracting [==================================================>]     145B/145B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  c5f3a9d4fd2b Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Extracting [==================================================>]     142B/142B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Extracting [==================================================>]     142B/142B
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  f23b28cea2a0 Pull complete
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  student02-ep11server Pulled
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Network compose_default  Creating
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Network compose_default  Created
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Container compose-student02-ep11server-1  Creating
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Container compose-student02-ep11server-1  Created
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Container compose-student02-ep11server-1  Starting
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Container compose-student02-ep11server-1  Started
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module main    
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module ep11server 
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module util    
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module entry   
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module config  
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Setting log level [debug] for module keyprotect 
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.293] Service postgrestemplate not found            #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service ep11manager not found                 #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service remoteconfig not found                #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service domaintemplate not found              #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service connectiontemplate not found          #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service tls not found                         #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service clientconnectiontemplate not found    #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service basevoteridtemplate not found         #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service logging not found                     #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service recoverykeyseedtemplate not found     #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.294] Service directiamauthtemplate not found       #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.303] Starting GREP11 server []                     #033[36mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.303] TLS is enabled                                #033[36mmodule#033[0m=config
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.303] Creating new listener for *config.EP11CryptoOpts  #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] hostname:port=192.168.22.80:9001
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: server_idx=0
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::makeNewC16ClientStub: target_str=192.168.22.80:9001
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: Done.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 1
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 37
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Docker compose result:
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:   CONTAINER ID   IMAGE                           COMMAND                 CREATED        STATUS                  PORTS                                                  NAMES
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  5afe4af5b8a9   quay.io/gmoney23/grep11server   "/usr/bin/ep11server"   1 second ago   Up Less than a second   0.0.0.0:9876->9876/tcp, :::9876->9876/tcp, 50052/tcp   compose-student02-ep11server-1
      Sep 05 21:07:35 ubuntu2204 hpcr-container[1777]:  Container service completed successfully
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Finished Service that creates a set of containers.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Reached target Workload is up and running..
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 54
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Done.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: server_idx=0
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::makeNewC16ClientStub: target_str=192.168.22.80:9001
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16OpenAdapter: Done.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=10, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 1
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 37
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Starting Phase2 Catch Service...
      Sep 05 21:07:35 ubuntu2204 hpcr-catch-success[1777]:  VSI has started successfully.
      Sep 05 21:07:35 ubuntu2204 hpcr-catch-success[1777]:  HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 54
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Done.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=1, ap_id=10, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 2
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 46
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 438
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Done.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Entering ...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=1, ap_id=10, dom_id=22
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 2
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 58
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Finished Phase2 Catch Service.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Reached target Multi-User System.
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 254
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  [c16client][debug] c16client::c16Request: Done.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Starting Record Runlevel Change in UTMP...
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.340] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:   message repeated 6 times: [#033[36mINFO#033[0m[2023-09-05 21:07:35.340] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util]
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.341] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:   message repeated 5 times: [#033[36mINFO#033[0m[2023-09-05 21:07:35.341] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util]
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[37mDEBU#033[0m[2023-09-05 21:07:35.341] Creating service backing server for *config.EP11CryptoOpts  #033[37mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.341] Loading ep11crypto service                    #033[36mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 compose-student02-ep11server-1[1777]:  #033[36mINFO#033[0m[2023-09-05 21:07:35.341] GRPC Server listening on [::]:9876 with TLS enabled  #033[36mmodule#033[0m=entry
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  systemd-update-utmp-runlevel.service: Deactivated successfully.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Finished Record Runlevel Change in UTMP.
      Sep 05 21:07:35 ubuntu2204 systemd[1777]:  Startup finished in 28.361s (kernel) + 6.315s (userspace) = 34.677s.
      Sep 05 21:07:36 ubuntu2204 verify-disk-encryption[1777]:  HPL13000I: Verify LUKS Encryption
      Sep 05 21:07:36 ubuntu2204 systemd[1777]:  verify-disk-encryption-invoker.service: Deactivated successfully.
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value for disk-encrypt: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Executed cmd: ('lsblk', '-b', '-n', '-o', 'NAME,SIZE')
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Stdout: vda                                           107374182400
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  ├─vda1                                          4292870144
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  └─vda2                                        103079215104
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:    └─luks-c88abd09-279d-4108-b5df-6faba4dd318e 103062437888
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  vdb                                                 417792
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  List of volumes greater than or equal to 10GB are: ['/dev/vda']
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Updated Volumes list: ['/dev/vda2']
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,MOUNTPOINT')
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Stdout: vda2
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  └─luks-c88abd09-279d-4108-b5df-6faba4dd318e /
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Boot volume is /dev/vda2
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Volume /dev/vda2 has mount point /
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  List of mounted volumes are: ['/dev/vda2']
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Verifying the boot disk /dev/vda2 is encrypted or not
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,TYPE')
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Stdout: vda2                                        part
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  └─luks-c88abd09-279d-4108-b5df-6faba4dd318e crypt
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Executed cmd: ('cryptsetup', 'isLuks', '/dev/vda2')
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Executed cmd: ('cryptsetup', 'luksDump', '/dev/vda2')
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  Return value: 0
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  HPL13003I: Checked for mount point /, LUKS encryption with 1 key slot found
      Sep 05 21:07:37 ubuntu2204 verify-disk-encryption[1777]:  HPL13001I: Boot volume and all the mounted data volumes are encrypted
      Sep 05 21:07:37 ubuntu2204 systemd-networkd[1777]:  veth09c9325: Gained IPv6LL
      Sep 05 21:07:37 ubuntu2204 systemd-networkd[1777]:  br-35f22f910f6f: Gained IPv6LL
      ```

*Congratulations!* You have reached a significant milestone in the lab.  You have successfully configured and launched your HPVS 2.1.6 GREP11 Server. Now all that is left is to test its functionality with some sample GREP11 client code.  You will set that up on your Ubuntu KVM guest.  Click _Next_ at the bottom right of the page to continue.
