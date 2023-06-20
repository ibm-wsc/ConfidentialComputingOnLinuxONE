# Start the GREP11 Server as a Secure Execution-enabled, HPVS 2.1.5 guest

## launch the HPVS 2.1.5 GREP11 server

You will start this section from your login session on the RHEL host, and will soon be instructed to switch to your Ubuntu KVM guest session.
But until then, start from this familiar window or tab:

<img src="../../../images/RHELHost.png" width="351" height="216" >

This fancy command figures out the last two characters of your assigned userid and is used in other commands in this section, so that the lab instructions will work for everybody:

   ``` bash
   suffix=$(temp=$(whoami) && echo ${temp: -2})
   ```

You aren't going to change anything here since it's already been defined for you by the instructors, but you can display the KVM guest definition of your HPVS 2.1.5 GREP11 Server:

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
            <source file='/var/lib/libvirt/images/hpcr/student02/ibm-hyper-protect-container-runtime-23.6.1.qcow2'/>
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
      # HPL11 build:23.6.1 enabler:23.5.1
      # Tue Jun 20 21:07:32 UTC 2023
      # Machine Type/Plant/Serial: 8561/02/31A38
      # delete old root partition...
      # create new root partition...
      # encrypt root partition...
      # create root filesystem...
      # write OS to root disk...
      # decrypt user-data...
      2 token decrypted, 0 encrypted token ignored
      # run attestation...
      # set hostname...
      # finish root disk setup...
      # Tue Jun 20 21:07:57 UTC 2023
      # HPL11 build:23.6.1 enabler:23.5.1
      # HPL11099I: bootloader end
      hpcr-dnslookup[733]: HPL14000I: Network connectivity check completed successfully.
      hpcr-logging[993]: Configuring logging ...
      hpcr-logging[994]: Version [1.1.133]
      hpcr-logging[994]: Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      hpcr-logging[994]: HPL01010I: Logging has been setup successfully.
      hpcr-logging[993]: Logging has been configured
      hpcr-catch-success[1339]: VSI has started successfully.
      hpcr-catch-success[1339]: HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
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
      Jun 20 21:07:59 ubuntu2204 vpcnode[2143]: authentication probe
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Linux version 5.15.0-72-generic (buildd@bos02-s390x-006) (gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #79-Ubuntu SMP Tue Apr 18 17:08:32 UTC 2023 (Ubuntu 5.15.0-72.79-generic 5.15.98)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  setup: Linux is running under KVM in 64-bit mode
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  setup: Relocating AMODE31 section of size 0x00003000
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  setup: The maximum memory size is 3812MB
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  cpu: 2 configured CPUs, 0 standby CPUs
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Write protected kernel read-only data: 18048k
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Zone ranges:
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:    DMA      [mem 0x0000000000000000-0x000000007fffffff]
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:    Normal   [mem 0x0000000080000000-0x00000000ee3fffff]
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Movable zone start for each node
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Early memory node ranges
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:    node   0: [mem 0x0000000000000000-0x00000000ee3fffff]
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Initmem setup node 0 [mem 0x0000000000000000-0x00000000ee3fffff]
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  On node 0, zone Normal: 7168 pages in unavailable ranges
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  percpu: Embedded 31 pages/cpu s89600 r8192 d29184 u126976
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  pcpu-alloc: s89600 r8192 d29184 u126976 alloc=31*4096
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  pcpu-alloc: [0] 0 [0] 1 
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Built 1 zonelists, mobility grouping on.  Total pages: 960624
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Policy zone: Normal
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Kernel command line: panic=0 blacklist=virtio_rng swiotlb=262144 cloud-init=disabled console=ttyS0 printk.time=0 systemd.getty_auto=0 systemd.firstboot=0 module.sig_enforce=1 quiet loglevel=0 systemd.show_status=0
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Unknown kernel command line parameters "blacklist=virtio_rng cloud-init=disabled", will be passed to user space.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Dentry cache hash table entries: 524288 (order: 10, 4194304 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Inode-cache hash table entries: 262144 (order: 9, 2097152 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  mem auto-init: stack:off, heap alloc:on, heap free:off
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  software IO TLB: mapped [mem 0x000000005fffc000-0x000000007fffc000] (512MB)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Memory: 3268492K/3903488K available (11432K kernel code, 2692K rwdata, 6616K rodata, 4184K init, 1256K bss, 634996K reserved, 0K cma-reserved)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  SLUB: HWalign=256, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ftrace: allocating 33988 entries in 133 pages
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ftrace: allocated 133 pages with 3 groups
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  rcu: Hierarchical RCU implementation.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  rcu: #011RCU restricting CPUs from NR_CPUS=512 to nr_cpu_ids=2.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  #011Rude variant of Tasks RCU enabled.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  #011Tracing variant of Tasks RCU enabled.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=2
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NR_IRQS: 3, nr_irqs: 3, preallocated irqs: 3
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  clocksource: tod: mask: 0xffffffffffffffff max_cycles: 0x3b0a9be803b0a9, max_idle_ns: 1805497147909793 ns
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  random: crng init done
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Console: colour dummy device 80x25
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  printk: console [ttyS0] enabled
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  printk: console [ttysclp0] enabled
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Calibrating delay loop (skipped)... 24038.00 BogoMIPS preset
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  pid_max: default: 32768 minimum: 301
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  LSM: Security Framework initializing
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  landlock: Up and running.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Yama: becoming mindful.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  AppArmor: AppArmor initialized
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Mount-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Mountpoint-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  rcu: Hierarchical SRCU implementation.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  smp: Bringing up secondary CPUs ...
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  smp: Brought up 1 node, 2 CPUs
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  devtmpfs: initialized
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  futex hash table entries: 512 (order: 5, 131072 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_NETLINK/PF_ROUTE protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: initializing netlink subsys (disabled)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=2000 audit(1687295252.457:1): state=initialized audit_enabled=0 res=1
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Spectre V2 mitigation: etokens
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  HugeTLB registered 1.00 MiB page size, pre-allocated 0 pages
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  iommu: Default domain type: Translated 
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  iommu: DMA domain TLB invalidation policy: strict mode 
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  SCSI subsystem initialized
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NetLabel: Initializing
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NetLabel:  domain hash size = 128
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NetLabel:  unlabeled traffic allowed by default
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  zpci: PCI is not supported because CPU facilities 69 or 71 are not available
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  VFS: Disk quotas dquot_6.6.0
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  AppArmor: AppArmor Filesystem Enabled
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_INET protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  IP idents hash table entries: 65536 (order: 7, 524288 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  tcp_listen_portaddr_hash hash table entries: 2048 (order: 3, 32768 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  TCP established hash table entries: 32768 (order: 6, 262144 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  TCP bind hash table entries: 32768 (order: 7, 524288 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  TCP: Hash tables configured (established 32768 bind 32768)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  MPTCP token hash table entries: 4096 (order: 4, 98304 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  UDP hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  UDP-Lite hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_UNIX/PF_LOCAL protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_XDP protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Trying to unpack rootfs image as initramfs...
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  kvm-s390: SIE is not available
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  hypfs: The hardware system does not support hypfs
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Initialise system trusted keyrings
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type blacklist registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  workingset: timestamp_bits=45 max_order=20 bucket_order=0
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  zbud: loaded
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  squashfs: version 4.0 (2009/01/31) Phillip Lougher
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  fuse: init (API version 7.34)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  integrity: Platform Keyring initialized
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type asymmetric registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Asymmetric key parser 'x509' registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Block layer SCSI generic (bsg) driver version 0.4 loaded (major 249)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  io scheduler mq-deadline registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  hvc_iucv: The z/VM IUCV HVC device driver cannot be used without z/VM
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  loop: module loaded
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  tun: Universal TUN/TAP device driver, 1.6
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  device-mapper: core: CONFIG_IMA_DISABLE_HTABLE is disabled. Duplicate IMA measurements will not be recorded in the IMA log.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  device-mapper: uevent: version 1.0.3
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  device-mapper: ioctl: 4.45.0-ioctl (2021-03-22) initialised: dm-devel@redhat.com
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  drop_monitor: Initializing network drop monitor service
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_INET6 protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Freeing initrd memory: 9804K
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Segment Routing with IPv6
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  In-situ OAM (IOAM) with IPv6
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  NET: Registered PF_PACKET protocol family
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type dns_resolver registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  cio: Channel measurement facility initialized using format extended (mode autodetected)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  sclp_sd: Store Data request failed (eq=2, di=3, response=0x40f0, flags=0x00, status=0, rc=-5)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ap: The hardware system does not support AP instructions
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  registered taskstats version 1
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loading compiled-in X.509 certificates
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Build time autogenerated kernel key: d047798e752e344562b493d833c08c247d3dd996'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Live Patch Signing: 14df34d1a87cf37625abec039ef2bf521249b969'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Kernel Module Signing: 88f752e560a1e0737e31163a466ad7b70a850c19'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  blacklist: Loading compiled-in revocation X.509 certificates
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing: 61482aa2830d0ab2ad5af10b7250da9033ddcef0'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2017): 242ade75ac4a15e50d50c84b0d45ff3eae707a03'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (ESM 2018): 365188c1d374d6b07c3c8f240f8ef722433d6a8b'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2019): c0746fd6c5da3ae827864651ad66ae47fe24b3e8'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v1): a8d54bbb3825cfb94fa13c9f8a594a195c107b8d'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v2): 4cf046892d6fd3c9a5b03f98d845f90851dc6a8c'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v3): 100437bb6de6e469b581e61cd66bce3ef4ed53af'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (Ubuntu Core 2019): c1d57b8f6b743f23ee41f4f7ee292f06eecadfb9'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  zswap: loaded using pool lzo/zbud
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type .fscrypt registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type fscrypt-provisioning registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Key type encrypted registered
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  AppArmor: AppArmor sha1 policy hashing enabled
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ima: No TPM chip found, activating TPM-bypass!
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loading compiled-in module X.509 certificates
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Loaded X.509 cert 'Build time autogenerated kernel key: d047798e752e344562b493d833c08c247d3dd996'
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ima: Allocated hash algorithm: sha1
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ima: No architecture policies found
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: Initialising EVM extended attributes:
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.selinux
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.SMACK64
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.SMACK64EXEC
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.SMACK64TRANSMUTE
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.SMACK64MMAP
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.apparmor
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.ima
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: security.capability
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  evm: HMAC attrs: 0x1
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Freeing unused kernel image (initmem) memory: 4184K
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Write protected read-only-after-init data: 136k
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Checked W+X mappings: passed, no unexpected W+X pages found
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Run /init as init process
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:    with arguments:
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:      /init
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:    with environment:
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:      HOME=/
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:      TERM=linux
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:      blacklist=virtio_rng
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:      cloud-init=disabled
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  virtio_blk virtio0: [vda] 209715200 512-byte logical blocks (107 GB/100 GiB)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:   vda: vda1 vda2
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  virtio_blk virtio1: [vdb] 812 512-byte logical blocks (416 kB/406 KiB)
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ISO 9660 Extensions: Microsoft Joliet Level 3
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  ISO 9660 Extensions: RRIP_1991A
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  EXT4-fs (dm-0): re-mounted. Opts: (null). Quota mode: none.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  systemd 249.11-0ubuntu3.9 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT +GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY -P11KIT -QRENCODE +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified)
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Detected virtualization kvm.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Detected architecture s390x.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Hostname set to <student02-grep11server>.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Initializing machine ID from D-Bus machine ID.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Installed transient /etc/machine-id file.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  /lib/systemd/system/verify-disk-encryption-invoker.service:6: Special user nobody configured, this is not safe!
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  /lib/systemd/system/se-dnslookup.service:10: Special user nobody configured, this is not safe!
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  /lib/systemd/system/hpl-catch-success.service:13: Special user nobody configured, this is not safe!
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  /lib/systemd/system/hpl-catch-failed.service:10: Special user nobody configured, this is not safe!
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Queued start job for default target Multi-User System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Created slice Slice /system/modprobe.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Created slice Slice /system/systemd-fsck.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Created slice User and Session Slice.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Dispatch Password Requests to Console Directory Watch.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Forward Password Requests to Wall Directory Watch.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Set up automount Arbitrary Executable File Formats File System Automount Point.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Local Encrypted Volumes.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Path Units.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Remote File Systems.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Slice Units.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Swaps.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Local Verity Protected Volumes.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Syslog Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on fsck to fsckd communication Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on initctl Compatibility Named Pipe.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Journal Audit Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Journal Socket (/dev/log).
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Journal Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Network Service Netlink Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on udev Control Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on udev Kernel Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting Huge Pages File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting POSIX Message Queue File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting Kernel Debug File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting Kernel Trace File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Journal Service...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Set the console keyboard layout...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Create List of Static Device Nodes...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module chromeos_pstore...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module configfs...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module drm...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module efi_pstore...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module fuse...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module pstore_blk...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module pstore_zone...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Module ramoops...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in OpenVSwitch configuration for cleanup being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting File System Check on Root Device...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load Kernel Modules...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Coldplug All udev Devices...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted Huge Pages File System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted POSIX Message Queue File System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted Kernel Debug File System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted Kernel Trace File System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Create List of Static Device Nodes.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@chromeos_pstore.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module chromeos_pstore.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@configfs.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module configfs.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@efi_pstore.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module efi_pstore.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@pstore_blk.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module pstore_blk.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@ramoops.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module ramoops.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished File System Check on Root Device.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Modules.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting Kernel Configuration File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started File System Check Daemon to report status.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Remount Root and Kernel File Systems...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Apply Kernel Variables...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@fuse.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module fuse.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting FUSE Control File System...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted Kernel Configuration File System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted FUSE Control File System.
      Jun 20 21:08:00 ubuntu2204 systemd-journald[2143]:  Journal started
      Jun 20 21:08:00 ubuntu2204 systemd-journald[2143]:  Runtime Journal (/run/log/journal/9ca2df8bfe994479acbc520c6a201489) is 4.0M, max 32.0M, 28.0M free.
      Jun 20 21:08:00 ubuntu2204 systemd-fsck[2143]:  /dev/mapper/luks-fc0025ac-addb-4246-9b01-1e328a017ec5: clean, 26404/6291456 files, 805374/25161728 blocks
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Journal Service.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  EXT4-fs (dm-0): re-mounted. Opts: errors=remount-ro. Quota mode: none.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Remount Root and Kernel File Systems.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Flush Journal to Persistent Storage...
      Jun 20 21:08:00 ubuntu2204 systemd-journald[2143]:  Time spent on flushing to /var/log/journal/9ca2df8bfe994479acbc520c6a201489 is 1.486ms for 250 entries.
      Jun 20 21:08:00 ubuntu2204 systemd-journald[2143]:  System Journal (/var/log/journal/9ca2df8bfe994479acbc520c6a201489) is 8.0M, max 4.0G, 3.9G free.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load/Save Random Seed...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Create System Users...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@pstore_zone.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module pstore_zone.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in Platform Persistent Storage Archival being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Apply Kernel Variables.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Create System Users.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Create Static Device Nodes in /dev...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Coldplug All udev Devices.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load/Save Random Seed.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in First Boot Complete being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Create Static Device Nodes in /dev.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Rule-based Manager for Device Events and Files...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  modprobe@drm.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load Kernel Module drm.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Flush Journal to Persistent Storage.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Rule-based Manager for Device Events and Files.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Network Configuration...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Set the console keyboard layout.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Preparation for Local File Systems.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  VFIO - User Level meta-driver version: 0.3
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  lo: Link UP
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  lo: Gained carrier
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  Enumeration completed
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Network Configuration.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Wait for Network to be Configured...
      Jun 20 21:08:00 ubuntu2204 systemd-udevd[2143]:  Using default interface naming scheme 'v249'.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  virtio_net virtio2 enc1: renamed from eth0
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Wait for Network to be Configured.
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  eth0: Interface name change detected, renamed to enc1.
      Jun 20 21:08:00 ubuntu2204 systemd-udevd[2143]:  Using default interface naming scheme 'v249'.
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  enc1: Link UP
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Found device /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting File System Check on /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485...
      Jun 20 21:08:00 ubuntu2204 systemd-fsck[2143]:  /dev/vda1: clean, 13/262144 files, 138348/1048064 blocks
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished File System Check on /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounting /boot...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Mounted /boot.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Local File Systems.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Load AppArmor profiles...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Set console font and keymap...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in Set Up Additional Binary Formats being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in Store a System Token in an EFI Variable being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Commit a transient machine-id on disk...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Create Volatile Files and Directories...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Set console font and keymap.
      Jun 20 21:08:00 ubuntu2204 apparmor.systemd[2143]:  Restarting AppArmor
      Jun 20 21:08:00 ubuntu2204 apparmor.systemd[2143]:  Reloading AppArmor profiles
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Create Volatile Files and Directories.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Commit a transient machine-id on disk.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Network Name Resolution...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Network Time Synchronization...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Record System Boot/Shutdown in UTMP...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Record System Boot/Shutdown in UTMP.
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=717 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=717 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.077:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=717 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.077:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=717 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 systemd-resolved[2143]:  Positive Trust Anchors:
      Jun 20 21:08:00 ubuntu2204 systemd-resolved[2143]:  . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
      Jun 20 21:08:00 ubuntu2204 systemd-resolved[2143]:  Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172.in-addr.arpa 20.172.in-addr.arpa 21.172.in-addr.arpa 22.172.in-addr.arpa 23.172.in-addr.arpa 24.172.in-addr.arpa 25.172.in-addr.arpa 26.172.in-addr.arpa 27.172.in-addr.arpa 28.172.in-addr.arpa 29.172.in-addr.arpa 30.172.in-addr.arpa 31.172.in-addr.arpa 168.192.in-addr.arpa d.f.ip6.arpa corp home internal intranet lan local private test
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Network Time Synchronization.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target System Time Set.
      Jun 20 21:08:00 ubuntu2204 systemd-resolved[2143]:  Using system hostname 'student02-grep11server'.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Network Name Resolution.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Network.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Network is Online.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Host and Network Name Lookups.
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=716 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 apparmor.systemd[2143]:  Skipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.147:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=716 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Load AppArmor profiles.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target System Initialization.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Daily apt download activities.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Daily apt upgrade and clean activities.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Daily dpkg database backup timer.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Periodic ext4 Online Metadata Check for All Filesystems.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Discard unused blocks once a week.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Daily rotation of log files.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Message of the Day.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Podman auto-update timer.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Daily Cleanup of Temporary Directories.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Ubuntu Advantage Timer for running repeated jobs.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Timer for calling verify disk encryption invoker service.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Timer Units.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on D-Bus System Message Bus Socket.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.357:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.357:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.357:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.357:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=721 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Docker Socket for the API...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Podman API Socket.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Listening on Docker Socket for the API.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Socket Units.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Basic System.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting containerd container runtime...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Regular background program processing daemon.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started D-Bus System Message Bus.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Save initial kernel messages after boot.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Remove Stale Online ext4 Metadata Check Snapshots...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in getty on tty2-tty6 if dbus and logind are not available being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Login Prompts.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Dispatcher daemon for systemd-networkd...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Podman auto-update service...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Podman Start All Containers With Restart Policy Set To Always...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Podman API Service...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Logging Configuration...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting User Login Management...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Permit User Sessions...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in Ubuntu Advantage reboot cmds being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Condition check resulted in Ubuntu Pro Background Auto Attach being skipped.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Podman API Service.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Permit User Sessions.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Set console scheme...
      Jun 20 21:08:00 ubuntu2204 cron[2143]:  (CRON) INFO (pidfile fd = 3)
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Set console scheme.
      Jun 20 21:08:00 ubuntu2204 cron[2143]:  (CRON) INFO (Running @reboot jobs)
      Jun 20 21:08:00 ubuntu2204 dbus-daemon[2143]:  [system] AppArmor D-Bus mediation is enabled
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.479187933Z" level=info msg="starting containerd" revision= version="1.6.12-0ubuntu1~22.04.1"
      Jun 20 21:08:00 ubuntu2204 systemd-logind[2143]:  New seat seat0.
      Jun 20 21:08:00 ubuntu2204 networkd-dispatcher[2143]:  No valid path found for iwconfig
      Jun 20 21:08:00 ubuntu2204 networkd-dispatcher[2143]:  No valid path found for iw
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started User Login Management.
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.531800803Z" level=info msg="loading plugin \"io.containerd.content.v1.content\"..." type=io.containerd.content.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.531877170Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." type=io.containerd.snapshotter.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.536194535Z" level=info msg="skip loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." error="path /var/lib/containerd/io.containerd.snapshotter.v1.btrfs (ext4) must be a btrfs filesystem to be used with the btrfs snapshotter: skip plugin" type=io.containerd.snapshotter.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.536207792Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.native\"..." type=io.containerd.snapshotter.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.536260801Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.overlayfs\"..." type=io.containerd.snapshotter.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.536391317Z" level=info msg="loading plugin \"io.containerd.metadata.v1.bolt\"..." type=io.containerd.metadata.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.536425000Z" level=info msg="metadata content store policy set" policy=shared
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Dispatcher daemon for systemd-networkd.
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546477563Z" level=info msg="loading plugin \"io.containerd.differ.v1.walking\"..." type=io.containerd.differ.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546501232Z" level=info msg="loading plugin \"io.containerd.event.v1.exchange\"..." type=io.containerd.event.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546511155Z" level=info msg="loading plugin \"io.containerd.gc.v1.scheduler\"..." type=io.containerd.gc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546532988Z" level=info msg="loading plugin \"io.containerd.service.v1.introspection-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546547828Z" level=info msg="loading plugin \"io.containerd.service.v1.containers-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546558180Z" level=info msg="loading plugin \"io.containerd.service.v1.content-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546567912Z" level=info msg="loading plugin \"io.containerd.service.v1.diff-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546578430Z" level=info msg="loading plugin \"io.containerd.service.v1.images-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546591016Z" level=info msg="loading plugin \"io.containerd.service.v1.leases-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546663881Z" level=info msg="loading plugin \"io.containerd.service.v1.namespaces-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546674077Z" level=info msg="loading plugin \"io.containerd.service.v1.snapshots-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546684537Z" level=info msg="loading plugin \"io.containerd.runtime.v1.linux\"..." type=io.containerd.runtime.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546738421Z" level=info msg="loading plugin \"io.containerd.runtime.v2.task\"..." type=io.containerd.runtime.v2
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546788161Z" level=info msg="loading plugin \"io.containerd.monitor.v1.cgroups\"..." type=io.containerd.monitor.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546958318Z" level=info msg="loading plugin \"io.containerd.service.v1.tasks-service\"..." type=io.containerd.service.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546976096Z" level=info msg="loading plugin \"io.containerd.grpc.v1.introspection\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.546991432Z" level=info msg="loading plugin \"io.containerd.internal.v1.restart\"..." type=io.containerd.internal.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547018812Z" level=info msg="loading plugin \"io.containerd.grpc.v1.containers\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547029505Z" level=info msg="loading plugin \"io.containerd.grpc.v1.content\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547039529Z" level=info msg="loading plugin \"io.containerd.grpc.v1.diff\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547051966Z" level=info msg="loading plugin \"io.containerd.grpc.v1.events\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547062255Z" level=info msg="loading plugin \"io.containerd.grpc.v1.healthcheck\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547071601Z" level=info msg="loading plugin \"io.containerd.grpc.v1.images\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547080787Z" level=info msg="loading plugin \"io.containerd.grpc.v1.leases\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547090100Z" level=info msg="loading plugin \"io.containerd.grpc.v1.namespaces\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.547100265Z" level=info msg="loading plugin \"io.containerd.internal.v1.opt\"..." type=io.containerd.internal.v1
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  e2scrub_reap.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Remove Stale Online ext4 Metadata Check Snapshots.
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548562839Z" level=info msg="loading plugin \"io.containerd.grpc.v1.snapshots\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548577954Z" level=info msg="loading plugin \"io.containerd.grpc.v1.tasks\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548589250Z" level=info msg="loading plugin \"io.containerd.grpc.v1.version\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548598769Z" level=info msg="loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." type=io.containerd.tracing.processor.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548609935Z" level=info msg="skip loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." error="no OpenTelemetry endpoint: skip plugin" type=io.containerd.tracing.processor.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548622296Z" level=info msg="loading plugin \"io.containerd.internal.v1.tracing\"..." type=io.containerd.internal.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548639569Z" level=error msg="failed to initialize a tracing processor \"otlp\"" error="no OpenTelemetry endpoint: skip plugin"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548683343Z" level=info msg="loading plugin \"io.containerd.grpc.v1.cri\"..." type=io.containerd.grpc.v1
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548746753Z" level=info msg="Start cri plugin with config {PluginConfig:{ContainerdConfig:{Snapshotter:overlayfs DefaultRuntimeName:runc DefaultRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} UntrustedWorkloadRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} Runtimes:map[runc:{Type:io.containerd.runc.v2 Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[BinaryName: CriuImagePath: CriuPath: CriuWorkPath: IoGid:0 IoUid:0 NoNewKeyring:false NoPivotRoot:false Root: ShimCgroup: SystemdCgroup:false] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0}] NoPivot:false DisableSnapshotAnnotations:true DiscardUnpackedLayers:false IgnoreRdtNotEnabledErrors:false} CniConfig:{NetworkPluginBinDir:/opt/cni/bin NetworkPluginConfDir:/etc/cni/net.d NetworkPluginMaxConfNum:1 NetworkPluginConfTemplate: IPPreference:} Registry:{ConfigPath: Mirrors:map[] Configs:map[] Auths:map[] Headers:map[]} ImageDecryption:{KeyModel:node} DisableTCPService:true StreamServerAddress:127.0.0.1 StreamServerPort:0 StreamIdleTimeout:4h0m0s EnableSelinux:false SelinuxCategoryRange:1024 SandboxImage:registry.k8s.io/pause:3.6 StatsCollectPeriod:10 SystemdCgroup:false EnableTLSStreaming:false X509KeyPairStreaming:{TLSCertFile: TLSKeyFile:} MaxContainerLogLineSize:16384 DisableCgroup:false DisableApparmor:false RestrictOOMScoreAdj:false MaxConcurrentDownloads:3 DisableProcMount:false UnsetSeccompProfile: TolerateMissingHugetlbController:true DisableHugetlbController:true DeviceOwnershipFromSecurityContext:false IgnoreImageDefinedVolumes:false NetNSMountsUnderStateDir:false EnableUnprivilegedPorts:false EnableUnprivilegedICMP:false} ContainerdRootDir:/var/lib/containerd ContainerdEndpoint:/run/containerd/containerd.sock RootDir:/var/lib/containerd/io.containerd.grpc.v1.cri StateDir:/run/containerd/io.containerd.grpc.v1.cri}"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548789719Z" level=info msg="Connect containerd service"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.548811504Z" level=info msg="Get image filesystem path \"/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs\""
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549283739Z" level=info msg="Start subscribing containerd event"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549316570Z" level=info msg="Start recovering state"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549355068Z" level=info msg="Start event monitor"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549365779Z" level=info msg="Start snapshots syncer"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549372431Z" level=info msg="Start cni network conf syncer for default"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549381157Z" level=info msg="Start streaming server"
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549551235Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.549572232Z" level=info msg=serving... address=/run/containerd/containerd.sock
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started containerd container runtime.
      Jun 20 21:08:00 ubuntu2204 containerd[2143]:  time="2023-06-20T21:07:58.553142381Z" level=info msg="containerd successfully booted in 0.075942s"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="/usr/bin/podman filtering at log level info"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="/usr/bin/podman filtering at log level info"
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Docker Application Container Engine...
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.629235741Z" level=info msg="Starting up"
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.629578006Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so using resolv.conf: /run/systemd/resolve/resolv.conf"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="[graphdriver] using prior storage driver: overlay"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  2023-06-20 21:07:58.662805051 +0000 UTC m=+0.233984960 system refresh
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="Setting parallel job count to 7"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="using systemd socket activation to determine API endpoint"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="using API endpoint: ''"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="Setting parallel job count to 7"
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  time="2023-06-20T21:07:58Z" level=info msg="API service listening on \"/run/podman/podman.sock\""
      Jun 20 21:08:00 ubuntu2204 audit[2143]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=790 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.690303102Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.690339977Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.690374681Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.690402987Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.691359103Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.691394886Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.691423097Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.691451137Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  audit: type=1400 audit(1687295278.687:9): apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=790 comm="apparmor_parser"
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.716145996Z" level=info msg="Loading containers: start."
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  bridge: filtering via arp/ip/ip6tables is no longer available by default. Update your scripts to load br_netfilter if you need this.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Bridge firewalling registered
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  etc-machine\x2did.mount: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Podman Start All Containers With Restart Policy Set To Always.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  Initializing XFRM netlink socket
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.889618754Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
      Jun 20 21:08:00 ubuntu2204 systemd-udevd[2143]:  Using default interface naming scheme 'v249'.
      Jun 20 21:08:00 ubuntu2204 networkd-dispatcher[2143]:  WARNING:Unknown index 3 seen, reloading interface list
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  docker0: Link UP
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  podman-auto-update.service: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Podman auto-update service.
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.939029517Z" level=info msg="Loading containers: done."
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  enc1: Gained carrier
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  enc1: DHCPv4 address 172.16.0.64/24 via 172.16.0.1
      Jun 20 21:08:00 ubuntu2204 dbus-daemon[2143]:  [system] Activating via systemd: service name='org.freedesktop.hostname1' unit='dbus-org.freedesktop.hostname1.service' requested by ':1.0' (uid=100 pid=646 comm="/lib/systemd/systemd-networkd " label="unconfined")
      Jun 20 21:08:00 ubuntu2204 systemd-timesyncd[2143]:  Network configuration changed, trying to establish connection.
      Jun 20 21:08:00 ubuntu2204 kernel[2143]:  IPv6: ADDRCONF(NETDEV_CHANGE): enc1: link becomes ready
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.960940594Z" level=info msg="Docker daemon" commit="20.10.21-0ubuntu1~22.04.3" graphdriver(s)=overlay2 version=20.10.21
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:58.960988576Z" level=info msg="Daemon has completed initialization"
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Hostname Service...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Docker Application Container Engine.
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:07:59.010938714Z" level=info msg="API listen on /run/docker.sock"
      Jun 20 21:08:00 ubuntu2204 dbus-daemon[2143]:  [system] Successfully activated service 'org.freedesktop.hostname1'
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Hostname Service.
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  Could not set hostname: Access denied
      Jun 20 21:08:00 ubuntu2204 systemd-timesyncd[2143]:  Initial synchronization to time server 185.125.190.58:123 (ntp.ubuntu.com).
      Jun 20 21:08:00 ubuntu2204 hpcr-dnslookup[2143]:  HPL14000I: Network connectivity check completed successfully.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Logging Configuration.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Early Initialization.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Logging to remote monitoring server is initiated..
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Logging Configuration...
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  Configuring logging ...
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  Version [1.1.133]
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  ValidateContractE ...
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  config written: /etc/rsyslog.d/22-logging.conf
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  HPL01010I: Logging has been setup successfully.
      Jun 20 21:08:00 ubuntu2204 hpcr-logging[2143]:  Logging has been configured
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Logging Configuration.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting System Logging Service...
      Jun 20 21:08:00 ubuntu2204 rsyslogd[2143]:  rsyslogd's groupid changed to 111
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started System Logging Service.
      Jun 20 21:08:00 ubuntu2204 rsyslogd[2143]:  rsyslogd's userid changed to 104
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Synchronizes the Logging Target.
      Jun 20 21:08:00 ubuntu2204 rsyslogd[2143]:  [origin software="rsyslogd" swVersion="8.2112.0" x-pid="998" x-info="https://www.rsyslog.com"] start
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Logging to remote log server is initiated..
      Jun 20 21:08:00 ubuntu2204 rsyslogd[2143]:  imjournal: No statefile exists, /var/spool/rsyslog/journal_state will be created (ignore if this is first run): No such file or directory [v8.2112.0 try https://www.rsyslog.com/e/2040 ]
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that does validation of contract...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting HPCR Registry Authentication...
      Jun 20 21:08:00 ubuntu2204 hpcr-registry-auth[2143]:  Starting Registry Authentication ...
      Jun 20 21:08:00 ubuntu2204 hpcr-registry-auth[2143]:  Version [1.0.61]
      Jun 20 21:08:00 ubuntu2204 hpcr-registry-auth[2143]:  Writing auth config: /root/.docker/config.json
      Jun 20 21:08:00 ubuntu2204 hpcr-registry-auth[2143]:  Registry Authentication started
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished HPCR Registry Authentication.
      Jun 20 21:08:00 ubuntu2204 hpcr-contract[2143]:  Welcome to SE Contract Validator
      Jun 20 21:08:00 ubuntu2204 hpcr-contract[2143]:  Contract file passed is:  /var/hyperprotect/user-data.decrypted
      Jun 20 21:08:00 ubuntu2204 rsyslogd[2143]:  imjournal: journal files changed, reloading...  [v8.2112.0 try https://www.rsyslog.com/e/0 ]
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 hpcr-contract[2143]:  Contract file is valid.
      Jun 20 21:08:00 ubuntu2204 hpcr-contract[2143]:  Extracting workload from /var/hyperprotect/user-data.decrypted to /var/hyperprotect/workload-data.decrypted
      Jun 20 21:08:00 ubuntu2204 hpcr-contract[2143]:  Extraction completed
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Service that does validation of contract.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that does signature validation of Env Workload of contract...
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Welcome to SE ENV Workload Signature Validator
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Decrypted Contract file passed is:  /var/hyperprotect/workload-data.decrypted
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Encrypted Contract file passed is:  /var/hyperprotect/cidata/user-data
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Check Dependency params Public key and EnvWorkload signature
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Access Public key and EnvWorkload signature
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Create combined EnvWorkload contract content
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Verify signing key, signature and combined EnvWorkload contract
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Verified OK
      Jun 20 21:08:00 ubuntu2204 hpcr-signature[2143]:  Successfully verified contract with signature and signing key
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Service that does signature validation of Env Workload of contract.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Contract is unpacked and ready for consumption..
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that waits until the user devices are ready...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Set docker image policy...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Set podman image policy...
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Starting image service...
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Extracting image contract
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Successfully extracted Image contract
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Extracting container contract
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Checking for image with digest
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  No image for DCT verification
      Jun 20 21:08:00 ubuntu2204 hpcr-image[2143]:  Image service completed successfully
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Set docker image policy.
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  Waiting for devices ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  Version [1.0.87]
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  WaitForDevices input=[/var/hyperprotect/user-data.decrypted], timeout=[2023-06-20 21:22:59.862807655 +0000 UTC m=+900.016813717]
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  ParseContract ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  ValidateContract ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  MergeVolumes ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-standby[2143]:  Waiting for devices is completed
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Service that waits until the user devices are ready.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that mounts the data volumes after they are ready...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  Mounting volumes ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  Version [1.0.87]
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  MountVolumes input=[/var/hyperprotect/user-data.decrypted]
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  ParseContract ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  ValidateContract ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  MergeVolumes ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  Mounting volumes ...
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  Volume config ..
      Jun 20 21:08:00 ubuntu2204 hpcr-disk-mount[2143]:  HPL07003I: Mounting volumes done
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Service that mounts the data volumes after they are ready.
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Starting container service...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Validating contract...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Compose folder /data1/compose created
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Compose folder: /data1/compose
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Validation completed
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Parsing contract...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Reached target Data volumes are mounted ready to be used..
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Parsing of the Contract File completed successfully
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Extracting compose...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Extracting done...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Extracting the ENV Contents...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Writing new env file /data1/compose/.env ...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Reading existing env file /data1/compose/.env ...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that creates a set of containers...
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Service that verifies all disks are encrypted and logs output to systemd journal.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Started Service that periodically logs entry to trigger verify disk encryption service.
      Jun 20 21:08:00 ubuntu2204 verify-disk-encryption[2143]:  Verify disk encryption started...
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Extracting of environment contents done
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Check if docker is ready
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Getting image source signatures
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Copying blob sha256:66d62867ae2452322f4769f943913be00b22e73039d1902e8f785b9f49838193
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  docker-compose.yml file is present in the directory
      Jun 20 21:08:00 ubuntu2204 hpcr-container[2143]:  Starting workload containers...
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Copying config sha256:9eca761232387055827db0a9f2232f2635bc8c6d5f23ecfb39d34bb4ab0dca09
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Writing manifest to image destination
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Storing signatures
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Loaded image(s): k8s.gcr.io/pause:3.5
      Jun 20 21:08:00 ubuntu2204 podman[2143]:  2023-06-20 21:07:59.938544559 +0000 UTC m=+0.097327720 image loadfromarchive  /usr/local/se-image-play/pause.tar
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-image-play
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Jun 20 21:08:00 ubuntu2204 hpcr-image-play[2143]:  Version [1.1.94]
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:  pam_unix(sudo:session): session closed for user nobody
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Set podman image policy.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Starting Service that creates a set of containers...
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-container-play
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Jun 20 21:08:00 ubuntu2204 hpcr-container-play[2143]:  Version [1.1.99]
      Jun 20 21:08:00 ubuntu2204 sudo[2143]:  pam_unix(sudo:session): session closed for user nobody
      Jun 20 21:08:00 ubuntu2204 hpcr-container-play[2143]:  HPL15004I: The pod started successfully.
      Jun 20 21:08:00 ubuntu2204 hpcr-container-play[2143]:  HPL15006I: No pod definitions found.
      Jun 20 21:08:00 ubuntu2204 systemd[2143]:  Finished Service that creates a set of containers.
      Jun 20 21:08:00 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:08:00.183330303Z" level=warning msg="reference for unknown type: " digest="sha256:a864174faadc39650e61ca45d8a3ceb01ea88602cfe6f4bd4e35c48e60556900" remote="quay.io/gmoney23/grep11server@sha256:a864174faadc39650e61ca45d8a3ceb01ea88602cfe6f4bd4e35c48e60556900"
      Jun 20 21:08:00 ubuntu2204 systemd-networkd[2143]:  enc1: Gained IPv6LL
      Jun 20 21:08:02 ubuntu2204 networkd-dispatcher[2143]:  WARNING:Unknown index 4 seen, reloading interface list
      Jun 20 21:08:02 ubuntu2204 systemd-networkd[2143]:  br-eb3e6aeba0d0: Link UP
      Jun 20 21:08:02 ubuntu2204 systemd[2143]:  var-lib-docker-overlay2-38846ed710aba5054153d7f048dfc924c4b3e992cfc72e050e0e6a4477c87eff\x2dinit-merged.mount: Deactivated successfully.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  var-lib-docker-overlay2-38846ed710aba5054153d7f048dfc924c4b3e992cfc72e050e0e6a4477c87eff-merged.mount: Deactivated successfully.
      Jun 20 21:08:03 ubuntu2204 networkd-dispatcher[2143]:  WARNING:Unknown index 5 seen, reloading interface list
      Jun 20 21:08:03 ubuntu2204 systemd-networkd[2143]:  veth4e57f43: Link UP
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered blocking state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered disabled state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  device veth4e57f43 entered promiscuous mode
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered blocking state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered forwarding state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered disabled state
      Jun 20 21:08:03 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:08:03.032785290Z" level=info msg="No non-localhost DNS nameservers are left in resolv.conf. Using default external servers: [nameserver 8.8.8.8 nameserver 8.8.4.4]"
      Jun 20 21:08:03 ubuntu2204 dockerd[2143]:  time="2023-06-20T21:08:03.032949576Z" level=info msg="IPv6 enabled; Adding default IPv6 external servers: [nameserver 2001:4860:4860::8888 nameserver 2001:4860:4860::8844]"
      Jun 20 21:08:03 ubuntu2204 containerd[2143]:  time="2023-06-20T21:08:03.094499893Z" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runtime=io.containerd.runc.v2 type=io.containerd.event.v1
      Jun 20 21:08:03 ubuntu2204 containerd[2143]:  time="2023-06-20T21:08:03.094545100Z" level=info msg="loading plugin \"io.containerd.internal.v1.shutdown\"..." runtime=io.containerd.runc.v2 type=io.containerd.internal.v1
      Jun 20 21:08:03 ubuntu2204 containerd[2143]:  time="2023-06-20T21:08:03.094554983Z" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io.containerd.runc.v2 type=io.containerd.ttrpc.v1
      Jun 20 21:08:03 ubuntu2204 containerd[2143]:  time="2023-06-20T21:08:03.094674606Z" level=info msg="starting signal loop" namespace=moby path=/run/containerd/io.containerd.runtime.v2.task/moby/52344ba5a7d2165c9b40a379d6503efcef8916e90556f44f5d947e89a3fffd20 pid=1252 runtime=io.containerd.runc.v2
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Started libcontainer container 52344ba5a7d2165c9b40a379d6503efcef8916e90556f44f5d947e89a3fffd20.
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  eth0: renamed from veth2e05138
      Jun 20 21:08:03 ubuntu2204 systemd-networkd[2143]:  veth4e57f43: Gained carrier
      Jun 20 21:08:03 ubuntu2204 systemd-networkd[2143]:  br-eb3e6aeba0d0: Gained carrier
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  IPv6: ADDRCONF(NETDEV_CHANGE): veth4e57f43: link becomes ready
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered blocking state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  br-eb3e6aeba0d0: port 1(veth4e57f43) entered forwarding state
      Jun 20 21:08:03 ubuntu2204 kernel[2143]:  IPv6: ADDRCONF(NETDEV_CHANGE): br-eb3e6aeba0d0: link becomes ready
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  dmesg.service: Deactivated successfully.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.487] Setting log level [debug] for module keyprotect 
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.487] Setting log level [debug] for module entry   
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.487] Setting log level [debug] for module util    
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Setting log level [debug] for module ep11server 
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Setting log level [debug] for module main    
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Setting log level [debug] for module config  
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Service recoverykeyseedtemplate not found     #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Service ep11manager not found                 #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Service domaintemplate not found              #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Service remoteconfig not found                #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.488] Service clientconnectiontemplate not found    #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.489] Service directiamauthtemplate not found       #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Docker Compose Logs:
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:   student02-ep11server Pulling
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Pulling fs layer
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Waiting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Downloading [>                                                  ]  190.8kB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Downloading [>                                                  ]   1.98kB/113.9kB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Downloading [====================>                              ]  7.829MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Downloading [==================================================>]  113.9kB/113.9kB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Downloading [==================================>                ]  12.94MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Downloading [=================================================> ]  18.41MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Downloading [>                                                  ]  73.09kB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Downloading [================================>                  ]     586B/890B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Downloading [==================================================>]     890B/890B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Extracting [==================================================>]     890B/890B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Extracting [==================================================>]     890B/890B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  620e494ced91 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Extracting [==============>                                    ]  32.77kB/113.9kB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Downloading [=========>                                         ]  1.276MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Extracting [==================================================>]  113.9kB/113.9kB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Extracting [==================================================>]  113.9kB/113.9kB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6c8a4d0d91d5 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Extracting [>                                                  ]  196.6kB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Downloading [===================================>               ]  4.929MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Downloading [==================================================>]     174B/174B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Extracting [========>                                          ]  3.146MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Downloading [=================================================> ]  6.973MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Downloading [==================================================>]     167B/167B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Extracting [================>                                  ]  6.095MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Extracting [========================>                          ]  9.241MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Downloading [==================================================>]     156B/156B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Extracting [==================================================>]  18.69MB/18.69MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  870d2d701868 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Extracting [>                                                  ]   98.3kB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Downloading [==================================================>]     139B/139B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Extracting [==============>                                    ]  2.064MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Extracting [=======================================>           ]  5.505MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Extracting [==================================================>]  6.994MB/6.994MB
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  17cad8585d31 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Extracting [==================================================>]     167B/167B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Extracting [==================================================>]     167B/167B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Downloading [==================================================>]     145B/145B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  957151557b52 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Extracting [==================================================>]     174B/174B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Extracting [==================================================>]     174B/174B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  356d6d7e116b Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Extracting [==================================================>]     156B/156B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Extracting [==================================================>]     156B/156B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  e90fb9a2f971 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Extracting [==================================================>]     139B/139B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Extracting [==================================================>]     139B/139B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  6792527a66d0 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Extracting [==================================================>]     145B/145B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Extracting [==================================================>]     145B/145B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  c5f3a9d4fd2b Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Downloading [==================================================>]     142B/142B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Verifying Checksum
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Download complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Extracting [==================================================>]     142B/142B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Extracting [==================================================>]     142B/142B
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  f23b28cea2a0 Pull complete
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  student02-ep11server Pulled
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Network compose_default  Creating
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Network compose_default  Created
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Container compose-student02-ep11server-1  Creating
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Container compose-student02-ep11server-1  Created
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Container compose-student02-ep11server-1  Starting
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Container compose-student02-ep11server-1  Started
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.491] Service logging not found                     #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.491] Service basevoteridtemplate not found         #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.491] Service connectiontemplate not found          #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.491] Service postgrestemplate not found            #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.491] Service tls not found                         #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.510] Starting GREP11 server []                     #033[36mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.510] TLS is enabled                                #033[36mmodule#033[0m=config
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.510] Creating new listener for *config.EP11CryptoOpts  #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] hostname:port=192.168.22.80:9001
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: server_idx=0
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::makeNewC16ClientStub: target_str=192.168.22.80:9001
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 1
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 37
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 54
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: server_idx=0
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::makeNewC16ClientStub: target_str=192.168.22.80:9001
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16OpenAdapter: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=10, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 1
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 37
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Docker compose result:
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:   CONTAINER ID   IMAGE                           COMMAND                 CREATED        STATUS                  PORTS                                                  NAMES
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  52344ba5a7d2   quay.io/gmoney23/grep11server   "/usr/bin/ep11server"   1 second ago   Up Less than a second   0.0.0.0:9876->9876/tcp, :::9876->9876/tcp, 50052/tcp   compose-student02-ep11server-1
      Jun 20 21:08:03 ubuntu2204 hpcr-container[2143]:  Container service completed successfully
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Finished Service that creates a set of containers.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Reached target Workload is up and running..
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 54
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=1, ap_id=10, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 2
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 46
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 438
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Entering ...
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=0, ap_id=8, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16DoEP11Request: Checking target i=1, ap_id=10, dom_id=22
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target still on same server
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client:c16DoEP11Request: Target list check passed. (server_idx=0)
      Jun 20 21:08:03 ubuntu2204 hpcr-catch-success[2143]:  VSI has started successfully.
      Jun 20 21:08:03 ubuntu2204 hpcr-catch-success[2143]:  HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: targets_num: 2
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: req_len: 58
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] C16ClientStub::DoRequest: Setting resp_len: 254
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  [c16client][debug] c16client::c16Request: Done.
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.555] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:   message repeated 4 times: [#033[36mINFO#033[0m[2023-06-20 21:08:03.555] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util]
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.556] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:   message repeated 7 times: [#033[36mINFO#033[0m[2023-06-20 21:08:03.556] admin.ep11.go:ep11server.(*CryptoServer).adminCommand:53: m_admin returned an error  #033[36merror code#033[0m=CKR_IBM_TARGET_INVALID #033[36mmodule#033[0m=util]
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[37mDEBU#033[0m[2023-06-20 21:08:03.556] Creating service backing server for *config.EP11CryptoOpts  #033[37mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.556] Loading ep11crypto service                    #033[36mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 compose-student02-ep11server-1[2143]:  #033[36mINFO#033[0m[2023-06-20 21:08:03.556] GRPC Server listening on [::]:9876 with TLS enabled  #033[36mmodule#033[0m=entry
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Starting Phase2 Catch Service...
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Finished Phase2 Catch Service.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Reached target Multi-User System.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Starting Record Runlevel Change in UTMP...
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  systemd-update-utmp-runlevel.service: Deactivated successfully.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Finished Record Runlevel Change in UTMP.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  Startup finished in 26.224s (kernel) + 5.975s (userspace) = 32.199s.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 21:08:03 ubuntu2204 systemd[2143]:  podman.service: Deactivated successfully.
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  HPL13000I: Verify LUKS Encryption
      Jun 20 21:08:05 ubuntu2204 systemd[2143]:  verify-disk-encryption-invoker.service: Deactivated successfully.
      Jun 20 21:08:05 ubuntu2204 systemd-networkd[2143]:  veth4e57f43: Gained IPv6LL
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value for disk-encrypt: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Executed cmd: ('lsblk', '-b', '-n', '-o', 'NAME,SIZE')
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Stdout: vda                                           107374182400
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  ├─vda1                                          4292870144
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  └─vda2                                        103079215104
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:    └─luks-fc0025ac-addb-4246-9b01-1e328a017ec5 103062437888
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  vdb                                                 415744
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  List of volumes greater than or equal to 10GB are: ['/dev/vda']
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Updated Volumes list: ['/dev/vda2']
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,MOUNTPOINT')
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Stdout: vda2
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  └─luks-fc0025ac-addb-4246-9b01-1e328a017ec5 /
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Boot volume is /dev/vda2
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Volume /dev/vda2 has mount point /
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  List of mounted volumes are: ['/dev/vda2']
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Verifying the boot disk /dev/vda2 is encrypted or not
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,TYPE')
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Stdout: vda2                                        part
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  └─luks-fc0025ac-addb-4246-9b01-1e328a017ec5 crypt
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Executed cmd: ('cryptsetup', 'isLuks', '/dev/vda2')
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Executed cmd: ('cryptsetup', 'luksDump', '/dev/vda2')
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Return value: 0
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  Checked for mount point /, LUKS encryption with 1 key slot found
      Jun 20 21:08:05 ubuntu2204 verify-disk-encryption[2143]:  HPL13001I: Boot volume and all the mounted data volumes are encrypted
      Jun 20 21:08:05 ubuntu2204 systemd-networkd[2143]:  br-eb3e6aeba0d0: Gained IPv6LL
      ```

*Congratulations!* You have reached a significant milestone in the lab.  You have successfully configured and launched your HPVS 2.1.5 GREP11 Server. Now all that is left is to test its functionality with some sample GREP11 client code.  You will set that up on your Ubuntu KVM guest.  Click _Next_ at the bottom right of the page to continue.
