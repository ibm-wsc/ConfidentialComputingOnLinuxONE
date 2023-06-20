# Launch HPVS guest for PayNow


You will start this section from your login session on the RHEL host. Start from this familiar window or tab:

<img src="../../../images/RHELHost.png" width="351" height="216" >

## launch the HPVS 2.1.5 KVM guest

This fancy command figures out (and displays) the last two characters of your assigned userid and is used in other commands in this section, so that the lab instructions will work for everybody:

   ``` bash
   suffix=$(temp=$(whoami) && echo ${temp: -2}) \
   && echo Your student suffix is ${suffix}

   ```

You aren't going to change anything here since it's already been defined for you by the instructors, but you can display the definition of your HPVS 2.1.5 KVM guest for the PayNow demo:

   ``` bash
   sudo virsh dumpxml paynowse${suffix}
   ```

???- example "Definition of HPVS KVM guest for PayNow Demo"

      ```
      <domain type='kvm'>
        <name>paynowse04</name>
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
            <source file='/var/lib/libvirt/images/labs/paynow/student04/ibm-hyper-protect-container-runtime-23.6.1.qcow2'/>
            <backingStore/>
            <target dev='vda' bus='virtio'/>
            <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0000'/>
          </disk>
          <disk type='file' device='disk'>
            <driver name='qemu' type='raw' cache='none' io='native' iommu='on'/>
            <source file='/var/lib/libvirt/images/labs/paynow/student04/ciiso.iso'/>
            <target dev='vdc' bus='virtio'/>
            <readonly/>
            <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0002'/>
          </disk>
          <controller type='pci' index='0' model='pci-root'/>
          <interface type='network'>
            <mac address='52:54:00:fc:6c:a8'/>
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

Start your HPVS Guest for the PayNow Demo and attach to its console.  Watch the messages carefully.  You should not see any failures:

   ``` bash
   sudo virsh start paynowse${suffix} --console
   
   ```

???- example "This is what success looks like"

      ```
      Domain 'paynowse04' started
      Connected to domain 'paynowse04'
      Escape character is ^] (Ctrl + ])
      # HPL11 build:23.6.1 enabler:23.5.1
      # Tue Jun 20 22:19:36 UTC 2023
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
      # Tue Jun 20 22:20:02 UTC 2023
      # HPL11 build:23.6.1 enabler:23.5.1
      # HPL11099I: bootloader end
      hpcr-dnslookup[817]: HPL14000I: Network connectivity check completed successfully.
      hpcr-logging[833]: Configuring logging ...
      hpcr-logging[834]: Version [1.1.133]
      hpcr-logging[834]: Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      hpcr-logging[834]: HPL01010I: Logging has been setup successfully.
      hpcr-logging[833]: Logging has been configured
      hpcr-catch-success[1375]: VSI has started successfully.
      hpcr-catch-success[1375]: HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
      ```

You will have to enter the `Ctrl + ]` key-combination to break out of the console.

## verify that messages from your HPVS KVM guest are received by rsyslog

**The logging of the HPVS KVM guest is going to the _rsyslog_ service that you configured on your Ubuntu guest, so switch to the terminal tab or window for your KVM standard guest.**

You should still be comfortably logged in on this tab or window:

<img src="../../../images/KVMGuest.png" width="351" height="217" />

The arguments to the _journalctl_ command here aren't the most elegant in the world, but, unless midnight passed since you started your HPVS KVM guest for PayNow, you will be able to see messages in rsyslog from when you just started up your HPVS KVM guest:

   ``` bash
   journalctl --since today --no-pager
   ```

There are a lot of messages logged, a veritable trove of treasure for the curious.  Here is an example of what you should be able to see:

???- example "Log messages in rsyslog from starting your HPVS KVM guest for the PayNow demo"

      ```
      Jun 20 22:20:05 ubuntu2204 vpcnode[23409]: authentication probe
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Linux version 5.15.0-72-generic (buildd@bos02-s390x-006) (gcc (Ubuntu 11.3.0-1ubuntu1~22.04.1) 11.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #79-Ubuntu SMP Tue Apr 18 17:08:32 UTC 2023 (Ubuntu 5.15.0-72.79-generic 5.15.98)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  setup: Linux is running under KVM in 64-bit mode
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  setup: Relocating AMODE31 section of size 0x00003000
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  setup: The maximum memory size is 3812MB
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  cpu: 2 configured CPUs, 0 standby CPUs
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Write protected kernel read-only data: 18048k
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Zone ranges:
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:    DMA      [mem 0x0000000000000000-0x000000007fffffff]
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:    Normal   [mem 0x0000000080000000-0x00000000ee3fffff]
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Movable zone start for each node
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Early memory node ranges
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:    node   0: [mem 0x0000000000000000-0x00000000ee3fffff]
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Initmem setup node 0 [mem 0x0000000000000000-0x00000000ee3fffff]
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  On node 0, zone Normal: 7168 pages in unavailable ranges
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  percpu: Embedded 31 pages/cpu s89600 r8192 d29184 u126976
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  pcpu-alloc: s89600 r8192 d29184 u126976 alloc=31*4096
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  pcpu-alloc: [0] 0 [0] 1 
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Built 1 zonelists, mobility grouping on.  Total pages: 960624
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Policy zone: Normal
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Kernel command line: panic=0 blacklist=virtio_rng swiotlb=262144 cloud-init=disabled console=ttyS0 printk.time=0 systemd.getty_auto=0 systemd.firstboot=0 module.sig_enforce=1 quiet loglevel=0 systemd.show_status=0
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Unknown kernel command line parameters "blacklist=virtio_rng cloud-init=disabled", will be passed to user space.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Dentry cache hash table entries: 524288 (order: 10, 4194304 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Inode-cache hash table entries: 262144 (order: 9, 2097152 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  mem auto-init: stack:off, heap alloc:on, heap free:off
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  software IO TLB: mapped [mem 0x0000000042dd0000-0x0000000062dd0000] (512MB)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Memory: 3268492K/3903488K available (11432K kernel code, 2692K rwdata, 6616K rodata, 4184K init, 1256K bss, 634996K reserved, 0K cma-reserved)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  SLUB: HWalign=256, Order=0-3, MinObjects=0, CPUs=2, Nodes=1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ftrace: allocating 33988 entries in 133 pages
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ftrace: allocated 133 pages with 3 groups
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  rcu: Hierarchical RCU implementation.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  rcu: #011RCU restricting CPUs from NR_CPUS=512 to nr_cpu_ids=2.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  #011Rude variant of Tasks RCU enabled.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  #011Tracing variant of Tasks RCU enabled.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  rcu: RCU calculated value of scheduler-enlistment delay is 10 jiffies.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  rcu: Adjusting geometry for rcu_fanout_leaf=16, nr_cpu_ids=2
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NR_IRQS: 3, nr_irqs: 3, preallocated irqs: 3
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  clocksource: tod: mask: 0xffffffffffffffff max_cycles: 0x3b0a9be803b0a9, max_idle_ns: 1805497147909793 ns
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  random: crng init done
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Console: colour dummy device 80x25
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  printk: console [ttyS0] enabled
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  printk: console [ttysclp0] enabled
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Calibrating delay loop (skipped)... 24038.00 BogoMIPS preset
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  pid_max: default: 32768 minimum: 301
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  LSM: Security Framework initializing
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  landlock: Up and running.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Yama: becoming mindful.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  AppArmor: AppArmor initialized
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Mount-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Mountpoint-cache hash table entries: 8192 (order: 4, 65536 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  rcu: Hierarchical SRCU implementation.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  smp: Bringing up secondary CPUs ...
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  smp: Brought up 1 node, 2 CPUs
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  devtmpfs: initialized
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  futex hash table entries: 512 (order: 5, 131072 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_NETLINK/PF_ROUTE protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: initializing netlink subsys (disabled)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=2000 audit(1687299576.735:1): state=initialized audit_enabled=0 res=1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Spectre V2 mitigation: etokens
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  HugeTLB registered 1.00 MiB page size, pre-allocated 0 pages
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  iommu: Default domain type: Translated 
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  iommu: DMA domain TLB invalidation policy: strict mode 
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  SCSI subsystem initialized
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NetLabel: Initializing
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NetLabel:  domain hash size = 128
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NetLabel:  unlabeled traffic allowed by default
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  zpci: PCI is not supported because CPU facilities 69 or 71 are not available
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  VFS: Disk quotas dquot_6.6.0
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  AppArmor: AppArmor Filesystem Enabled
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_INET protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  IP idents hash table entries: 65536 (order: 7, 524288 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  tcp_listen_portaddr_hash hash table entries: 2048 (order: 3, 32768 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  TCP established hash table entries: 32768 (order: 6, 262144 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  TCP bind hash table entries: 32768 (order: 7, 524288 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  TCP: Hash tables configured (established 32768 bind 32768)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  MPTCP token hash table entries: 4096 (order: 4, 98304 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  UDP hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  UDP-Lite hash table entries: 2048 (order: 4, 65536 bytes, linear)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_UNIX/PF_LOCAL protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_XDP protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Trying to unpack rootfs image as initramfs...
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  kvm-s390: SIE is not available
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  hypfs: The hardware system does not support hypfs
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Initialise system trusted keyrings
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type blacklist registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  workingset: timestamp_bits=45 max_order=20 bucket_order=0
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  zbud: loaded
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  squashfs: version 4.0 (2009/01/31) Phillip Lougher
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  fuse: init (API version 7.34)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  integrity: Platform Keyring initialized
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type asymmetric registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Asymmetric key parser 'x509' registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Block layer SCSI generic (bsg) driver version 0.4 loaded (major 249)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  io scheduler mq-deadline registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  hvc_iucv: The z/VM IUCV HVC device driver cannot be used without z/VM
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  loop: module loaded
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  tun: Universal TUN/TAP device driver, 1.6
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  device-mapper: core: CONFIG_IMA_DISABLE_HTABLE is disabled. Duplicate IMA measurements will not be recorded in the IMA log.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  device-mapper: uevent: version 1.0.3
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  device-mapper: ioctl: 4.45.0-ioctl (2021-03-22) initialised: dm-devel@redhat.com
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  drop_monitor: Initializing network drop monitor service
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_INET6 protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Freeing initrd memory: 9804K
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Segment Routing with IPv6
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  In-situ OAM (IOAM) with IPv6
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  NET: Registered PF_PACKET protocol family
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type dns_resolver registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  cio: Channel measurement facility initialized using format extended (mode autodetected)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  sclp_sd: Store Data request failed (eq=2, di=3, response=0x40f0, flags=0x00, status=0, rc=-5)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ap: The hardware system does not support AP instructions
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  registered taskstats version 1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loading compiled-in X.509 certificates
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Build time autogenerated kernel key: d047798e752e344562b493d833c08c247d3dd996'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Live Patch Signing: 14df34d1a87cf37625abec039ef2bf521249b969'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Kernel Module Signing: 88f752e560a1e0737e31163a466ad7b70a850c19'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  blacklist: Loading compiled-in revocation X.509 certificates
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing: 61482aa2830d0ab2ad5af10b7250da9033ddcef0'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2017): 242ade75ac4a15e50d50c84b0d45ff3eae707a03'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (ESM 2018): 365188c1d374d6b07c3c8f240f8ef722433d6a8b'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2019): c0746fd6c5da3ae827864651ad66ae47fe24b3e8'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v1): a8d54bbb3825cfb94fa13c9f8a594a195c107b8d'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v2): 4cf046892d6fd3c9a5b03f98d845f90851dc6a8c'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (2021 v3): 100437bb6de6e469b581e61cd66bce3ef4ed53af'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Canonical Ltd. Secure Boot Signing (Ubuntu Core 2019): c1d57b8f6b743f23ee41f4f7ee292f06eecadfb9'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  zswap: loaded using pool lzo/zbud
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type .fscrypt registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type fscrypt-provisioning registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Key type encrypted registered
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  AppArmor: AppArmor sha1 policy hashing enabled
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ima: No TPM chip found, activating TPM-bypass!
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loading compiled-in module X.509 certificates
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Loaded X.509 cert 'Build time autogenerated kernel key: d047798e752e344562b493d833c08c247d3dd996'
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ima: Allocated hash algorithm: sha1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ima: No architecture policies found
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: Initialising EVM extended attributes:
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.selinux
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.SMACK64
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.SMACK64EXEC
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.SMACK64TRANSMUTE
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.SMACK64MMAP
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.apparmor
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.ima
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: security.capability
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  evm: HMAC attrs: 0x1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Freeing unused kernel image (initmem) memory: 4184K
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Write protected read-only-after-init data: 136k
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Checked W+X mappings: passed, no unexpected W+X pages found
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Run /init as init process
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:    with arguments:
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:      /init
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:    with environment:
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:      HOME=/
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:      TERM=linux
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:      blacklist=virtio_rng
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:      cloud-init=disabled
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  virtio_blk virtio0: [vda] 209715200 512-byte logical blocks (107 GB/100 GiB)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  GPT:Primary header thinks Alt. header is not at the end of the disk.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  GPT:8388607 != 209715199
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  GPT:Alternate GPT header not at the end of the disk.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  GPT:8388607 != 209715199
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  GPT: Use GNU Parted to correct GPT errors.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:   vda: vda1
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  virtio_blk virtio1: [vdb] 760 512-byte logical blocks (389 kB/380 KiB)
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  EXT4-fs (dm-0): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ISO 9660 Extensions: Microsoft Joliet Level 3
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  ISO 9660 Extensions: RRIP_1991A
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  EXT4-fs (dm-0): re-mounted. Opts: (null). Quota mode: none.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  systemd 249.11-0ubuntu3.9 running in system mode (+PAM +AUDIT +SELINUX +APPARMOR +IMA +SMACK +SECCOMP +GCRYPT +GNUTLS +OPENSSL +ACL +BLKID +CURL +ELFUTILS +FIDO2 +IDN2 -IDN +IPTC +KMOD +LIBCRYPTSETUP +LIBFDISK +PCRE2 -PWQUALITY -P11KIT -QRENCODE +BZIP2 +LZ4 +XZ +ZLIB +ZSTD -XKBCOMMON +UTMP +SYSVINIT default-hierarchy=unified)
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Detected virtualization kvm.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Detected architecture s390x.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Hostname set to <student04-paynowdemo>.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Initializing machine ID from D-Bus machine ID.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Installed transient /etc/machine-id file.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  /lib/systemd/system/verify-disk-encryption-invoker.service:6: Special user nobody configured, this is not safe!
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  /lib/systemd/system/se-dnslookup.service:10: Special user nobody configured, this is not safe!
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  /lib/systemd/system/hpl-catch-success.service:13: Special user nobody configured, this is not safe!
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  /lib/systemd/system/hpl-catch-failed.service:10: Special user nobody configured, this is not safe!
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Queued start job for default target Multi-User System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Created slice Slice /system/modprobe.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Created slice Slice /system/systemd-fsck.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Created slice User and Session Slice.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Dispatch Password Requests to Console Directory Watch.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Forward Password Requests to Wall Directory Watch.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Set up automount Arbitrary Executable File Formats File System Automount Point.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Local Encrypted Volumes.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Path Units.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Remote File Systems.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Slice Units.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Swaps.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Local Verity Protected Volumes.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Syslog Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on fsck to fsckd communication Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on initctl Compatibility Named Pipe.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Journal Audit Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Journal Socket (/dev/log).
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Journal Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Network Service Netlink Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on udev Control Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on udev Kernel Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting Huge Pages File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting POSIX Message Queue File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting Kernel Debug File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting Kernel Trace File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Journal Service...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Set the console keyboard layout...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Create List of Static Device Nodes...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module chromeos_pstore...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module configfs...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module drm...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module efi_pstore...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module fuse...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module pstore_blk...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module pstore_zone...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Module ramoops...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in OpenVSwitch configuration for cleanup being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting File System Check on Root Device...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load Kernel Modules...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Coldplug All udev Devices...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted Huge Pages File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted POSIX Message Queue File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted Kernel Debug File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted Kernel Trace File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Create List of Static Device Nodes.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@chromeos_pstore.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module chromeos_pstore.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@configfs.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module configfs.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@efi_pstore.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module efi_pstore.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@fuse.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module fuse.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting FUSE Control File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting Kernel Configuration File System...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started File System Check Daemon to report status.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished File System Check on Root Device.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted FUSE Control File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted Kernel Configuration File System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Remount Root and Kernel File Systems...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Modules.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Apply Kernel Variables...
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  EXT4-fs (dm-0): re-mounted. Opts: errors=remount-ro. Quota mode: none.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Remount Root and Kernel File Systems.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load/Save Random Seed...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Create System Users...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Create System Users.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Create Static Device Nodes in /dev...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load/Save Random Seed.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in First Boot Complete being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Create Static Device Nodes in /dev.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Rule-based Manager for Device Events and Files...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@pstore_zone.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module pstore_zone.
      Jun 20 22:20:07 ubuntu2204 systemd-journald[23409]:  Journal started
      Jun 20 22:20:07 ubuntu2204 systemd-journald[23409]:  Runtime Journal (/run/log/journal/9ca2df8bfe994479acbc520c6a201489) is 4.0M, max 32.0M, 28.0M free.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Journal Service.
      Jun 20 22:20:07 ubuntu2204 systemd-journald[23409]:  Time spent on flushing to /var/log/journal/9ca2df8bfe994479acbc520c6a201489 is 1.516ms for 259 entries.
      Jun 20 22:20:07 ubuntu2204 systemd-journald[23409]:  System Journal (/var/log/journal/9ca2df8bfe994479acbc520c6a201489) is 8.0M, max 4.0G, 3.9G free.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Flush Journal to Persistent Storage...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Coldplug All udev Devices.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Apply Kernel Variables.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@ramoops.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module ramoops.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@pstore_blk.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module pstore_blk.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in Platform Persistent Storage Archival being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  modprobe@drm.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load Kernel Module drm.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Flush Journal to Persistent Storage.
      Jun 20 22:20:07 ubuntu2204 systemd-fsck[23409]:  /dev/mapper/luks-f151eb69-04c7-42d5-8859-105dc2fd8512: clean, 26404/6291456 files, 805371/25161728 blocks
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Rule-based Manager for Device Events and Files.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  VFIO - User Level meta-driver version: 0.3
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Network Configuration...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Set the console keyboard layout.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Preparation for Local File Systems.
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  lo: Link UP
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  lo: Gained carrier
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  Enumeration completed
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Network Configuration.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Wait for Network to be Configured...
      Jun 20 22:20:07 ubuntu2204 systemd-udevd[23409]:  Using default interface naming scheme 'v249'.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  virtio_net virtio2 enc1: renamed from eth0
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  eth0: Interface name change detected, renamed to enc1.
      Jun 20 22:20:07 ubuntu2204 systemd-udevd[23409]:  Using default interface naming scheme 'v249'.
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  enc1: Link UP
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Found device /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting File System Check on /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485...
      Jun 20 22:20:07 ubuntu2204 systemd-fsck[23409]:  /dev/vda1: clean, 13/262144 files, 138348/1048064 blocks
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished File System Check on /dev/disk/by-uuid/42d06a5d-e2c8-4de6-91cd-1bba75ce0485.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounting /boot...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Mounted /boot.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Local File Systems.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  EXT4-fs (vda1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Load AppArmor profiles...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Set console font and keymap...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in Set Up Additional Binary Formats being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in Store a System Token in an EFI Variable being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Commit a transient machine-id on disk...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Create Volatile Files and Directories...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Set console font and keymap.
      Jun 20 22:20:07 ubuntu2204 apparmor.systemd[23409]:  Restarting AppArmor
      Jun 20 22:20:07 ubuntu2204 apparmor.systemd[23409]:  Reloading AppArmor profiles
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Create Volatile Files and Directories.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Network Name Resolution...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Network Time Synchronization...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Record System Boot/Shutdown in UTMP...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Record System Boot/Shutdown in UTMP.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Commit a transient machine-id on disk.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Network Time Synchronization.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target System Time Set.
      Jun 20 22:20:07 ubuntu2204 systemd-resolved[23409]:  Positive Trust Anchors:
      Jun 20 22:20:07 ubuntu2204 systemd-resolved[23409]:  . IN DS 20326 8 2 e06d44b80b8f1d39a95c0b0d7c65d08458e880409bbc683457104237c7f8ec8d
      Jun 20 22:20:07 ubuntu2204 systemd-resolved[23409]:  Negative trust anchors: home.arpa 10.in-addr.arpa 16.172.in-addr.arpa 17.172.in-addr.arpa 18.172.in-addr.arpa 19.172.in-addr.arpa 20.172.in-addr.arpa 21.172.in-addr.arpa 22.172.in-addr.arpa 23.172.in-addr.arpa 24.172.in-addr.arpa 25.172.in-addr.arpa 26.172.in-addr.arpa 27.172.in-addr.arpa 28.172.in-addr.arpa 29.172.in-addr.arpa 30.172.in-addr.arpa 31.172.in-addr.arpa 168.192.in-addr.arpa d.f.ip6.arpa corp home internal intranet lan local private test
      Jun 20 22:20:07 ubuntu2204 systemd-resolved[23409]:  Using system hostname 'student04-paynowdemo'.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Network Name Resolution.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Network.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Host and Network Name Lookups.
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=720 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=720 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299602.955:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe" pid=720 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299602.955:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia_modprobe//kmod" pid=720 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=719 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 apparmor.systemd[23409]:  Skipping profile in /etc/apparmor.d/disable: usr.sbin.rsyslogd
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299602.975:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="lsb_release" pid=719 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Load AppArmor profiles.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target System Initialization.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Daily apt download activities.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Daily apt upgrade and clean activities.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Daily dpkg database backup timer.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Periodic ext4 Online Metadata Check for All Filesystems.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Discard unused blocks once a week.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Daily rotation of log files.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Message of the Day.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Podman auto-update timer.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Daily Cleanup of Temporary Directories.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Ubuntu Advantage Timer for running repeated jobs.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Timer for calling verify disk encryption invoker service.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Timer Units.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on D-Bus System Message Bus Socket.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299603.225:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-client.action" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299603.225:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/NetworkManager/nm-dhcp-helper" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299603.225:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/lib/connman/scripts/dhclient-script" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299603.225:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/{,usr/}sbin/dhclient" pid=721 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Docker Socket for the API...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Podman API Socket.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Listening on Docker Socket for the API.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Socket Units.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Basic System.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting containerd container runtime...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Regular background program processing daemon.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started D-Bus System Message Bus.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Save initial kernel messages after boot.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Remove Stale Online ext4 Metadata Check Snapshots...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in getty on tty2-tty6 if dbus and logind are not available being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Login Prompts.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Dispatcher daemon for systemd-networkd...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Podman Start All Containers With Restart Policy Set To Always...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Podman API Service...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting User Login Management...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Permit User Sessions...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in Ubuntu Advantage reboot cmds being skipped.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Podman API Service.
      Jun 20 22:20:07 ubuntu2204 cron[23409]:  (CRON) INFO (pidfile fd = 3)
      Jun 20 22:20:07 ubuntu2204 cron[23409]:  (CRON) INFO (Running @reboot jobs)
      Jun 20 22:20:07 ubuntu2204 dbus-daemon[23409]:  [system] AppArmor D-Bus mediation is enabled
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Permit User Sessions.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Set console scheme...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Set console scheme.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  e2scrub_reap.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Remove Stale Online ext4 Metadata Check Snapshots.
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.345079966Z" level=info msg="starting containerd" revision= version="1.6.12-0ubuntu1~22.04.1"
      Jun 20 22:20:07 ubuntu2204 systemd-logind[23409]:  New seat seat0.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started User Login Management.
      Jun 20 22:20:07 ubuntu2204 networkd-dispatcher[23409]:  No valid path found for iwconfig
      Jun 20 22:20:07 ubuntu2204 networkd-dispatcher[23409]:  No valid path found for iw
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="/usr/bin/podman filtering at log level info"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Dispatcher daemon for systemd-networkd.
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.373038358Z" level=info msg="loading plugin \"io.containerd.content.v1.content\"..." type=io.containerd.content.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374209251Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." type=io.containerd.snapshotter.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374342079Z" level=info msg="skip loading plugin \"io.containerd.snapshotter.v1.btrfs\"..." error="path /var/lib/containerd/io.containerd.snapshotter.v1.btrfs (ext4) must be a btrfs filesystem to be used with the btrfs snapshotter: skip plugin" type=io.containerd.snapshotter.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374378158Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.native\"..." type=io.containerd.snapshotter.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374447840Z" level=info msg="loading plugin \"io.containerd.snapshotter.v1.overlayfs\"..." type=io.containerd.snapshotter.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374619644Z" level=info msg="loading plugin \"io.containerd.metadata.v1.bolt\"..." type=io.containerd.metadata.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.374672754Z" level=info msg="metadata content store policy set" policy=shared
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383530055Z" level=info msg="loading plugin \"io.containerd.differ.v1.walking\"..." type=io.containerd.differ.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383549509Z" level=info msg="loading plugin \"io.containerd.event.v1.exchange\"..." type=io.containerd.event.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383560695Z" level=info msg="loading plugin \"io.containerd.gc.v1.scheduler\"..." type=io.containerd.gc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383585238Z" level=info msg="loading plugin \"io.containerd.service.v1.introspection-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383597323Z" level=info msg="loading plugin \"io.containerd.service.v1.containers-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383608375Z" level=info msg="loading plugin \"io.containerd.service.v1.content-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383620746Z" level=info msg="loading plugin \"io.containerd.service.v1.diff-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383632054Z" level=info msg="loading plugin \"io.containerd.service.v1.images-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383642830Z" level=info msg="loading plugin \"io.containerd.service.v1.leases-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383845563Z" level=info msg="loading plugin \"io.containerd.service.v1.namespaces-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383856350Z" level=info msg="loading plugin \"io.containerd.service.v1.snapshots-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383866684Z" level=info msg="loading plugin \"io.containerd.runtime.v1.linux\"..." type=io.containerd.runtime.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383916192Z" level=info msg="loading plugin \"io.containerd.runtime.v2.task\"..." type=io.containerd.runtime.v2
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.383980115Z" level=info msg="loading plugin \"io.containerd.monitor.v1.cgroups\"..." type=io.containerd.monitor.v1
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="/usr/bin/podman filtering at log level info"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384159889Z" level=info msg="loading plugin \"io.containerd.service.v1.tasks-service\"..." type=io.containerd.service.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384178977Z" level=info msg="loading plugin \"io.containerd.grpc.v1.introspection\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384189828Z" level=info msg="loading plugin \"io.containerd.internal.v1.restart\"..." type=io.containerd.internal.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384219340Z" level=info msg="loading plugin \"io.containerd.grpc.v1.containers\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384229411Z" level=info msg="loading plugin \"io.containerd.grpc.v1.content\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384239177Z" level=info msg="loading plugin \"io.containerd.grpc.v1.diff\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384248687Z" level=info msg="loading plugin \"io.containerd.grpc.v1.events\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384259800Z" level=info msg="loading plugin \"io.containerd.grpc.v1.healthcheck\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384269788Z" level=info msg="loading plugin \"io.containerd.grpc.v1.images\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384280622Z" level=info msg="loading plugin \"io.containerd.grpc.v1.leases\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384292828Z" level=info msg="loading plugin \"io.containerd.grpc.v1.namespaces\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.384303826Z" level=info msg="loading plugin \"io.containerd.internal.v1.opt\"..." type=io.containerd.internal.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.389981657Z" level=info msg="loading plugin \"io.containerd.grpc.v1.snapshots\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390004069Z" level=info msg="loading plugin \"io.containerd.grpc.v1.tasks\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390017200Z" level=info msg="loading plugin \"io.containerd.grpc.v1.version\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390027278Z" level=info msg="loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." type=io.containerd.tracing.processor.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390040350Z" level=info msg="skip loading plugin \"io.containerd.tracing.processor.v1.otlp\"..." error="no OpenTelemetry endpoint: skip plugin" type=io.containerd.tracing.processor.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390049827Z" level=info msg="loading plugin \"io.containerd.internal.v1.tracing\"..." type=io.containerd.internal.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390067105Z" level=error msg="failed to initialize a tracing processor \"otlp\"" error="no OpenTelemetry endpoint: skip plugin"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390098663Z" level=info msg="loading plugin \"io.containerd.grpc.v1.cri\"..." type=io.containerd.grpc.v1
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390162197Z" level=info msg="Start cri plugin with config {PluginConfig:{ContainerdConfig:{Snapshotter:overlayfs DefaultRuntimeName:runc DefaultRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} UntrustedWorkloadRuntime:{Type: Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0} Runtimes:map[runc:{Type:io.containerd.runc.v2 Path: Engine: PodAnnotations:[] ContainerAnnotations:[] Root: Options:map[BinaryName: CriuImagePath: CriuPath: CriuWorkPath: IoGid:0 IoUid:0 NoNewKeyring:false NoPivotRoot:false Root: ShimCgroup: SystemdCgroup:false] PrivilegedWithoutHostDevices:false BaseRuntimeSpec: NetworkPluginConfDir: NetworkPluginMaxConfNum:0}] NoPivot:false DisableSnapshotAnnotations:true DiscardUnpackedLayers:false IgnoreRdtNotEnabledErrors:false} CniConfig:{NetworkPluginBinDir:/opt/cni/bin NetworkPluginConfDir:/etc/cni/net.d NetworkPluginMaxConfNum:1 NetworkPluginConfTemplate: IPPreference:} Registry:{ConfigPath: Mirrors:map[] Configs:map[] Auths:map[] Headers:map[]} ImageDecryption:{KeyModel:node} DisableTCPService:true StreamServerAddress:127.0.0.1 StreamServerPort:0 StreamIdleTimeout:4h0m0s EnableSelinux:false SelinuxCategoryRange:1024 SandboxImage:registry.k8s.io/pause:3.6 StatsCollectPeriod:10 SystemdCgroup:false EnableTLSStreaming:false X509KeyPairStreaming:{TLSCertFile: TLSKeyFile:} MaxContainerLogLineSize:16384 DisableCgroup:false DisableApparmor:false RestrictOOMScoreAdj:false MaxConcurrentDownloads:3 DisableProcMount:false UnsetSeccompProfile: TolerateMissingHugetlbController:true DisableHugetlbController:true DeviceOwnershipFromSecurityContext:false IgnoreImageDefinedVolumes:false NetNSMountsUnderStateDir:false EnableUnprivilegedPorts:false EnableUnprivilegedICMP:false} ContainerdRootDir:/var/lib/containerd ContainerdEndpoint:/run/containerd/containerd.sock RootDir:/var/lib/containerd/io.containerd.grpc.v1.cri StateDir:/run/containerd/io.containerd.grpc.v1.cri}"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390207359Z" level=info msg="Connect containerd service"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.390230904Z" level=info msg="Get image filesystem path \"/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs\""
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.444038376Z" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.444065757Z" level=info msg=serving... address=/run/containerd/containerd.sock
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.444091654Z" level=info msg="containerd successfully booted in 0.112678s"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started containerd container runtime.
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446193808Z" level=info msg="Start subscribing containerd event"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446246259Z" level=info msg="Start recovering state"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446289473Z" level=info msg="Start event monitor"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446303192Z" level=info msg="Start snapshots syncer"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446311467Z" level=info msg="Start cni network conf syncer for default"
      Jun 20 22:20:07 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:03.446318574Z" level=info msg="Start streaming server"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="[graphdriver] using prior storage driver: overlay"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="Found CNI network podman (type=bridge) at /etc/cni/net.d/87-podman-bridge.conflist"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  2023-06-20 22:20:03.479504955 +0000 UTC m=+0.215839894 system refresh
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="Setting parallel job count to 7"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="using systemd socket activation to determine API endpoint"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="using API endpoint: ''"
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="API service listening on \"/run/podman/podman.sock\""
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  time="2023-06-20T22:20:03Z" level=info msg="Setting parallel job count to 7"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  etc-machine\x2did.mount: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Podman Start All Containers With Restart Policy Set To Always.
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  enc1: Gained carrier
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  IPv6: ADDRCONF(NETDEV_CHANGE): enc1: link becomes ready
      Jun 20 22:20:07 ubuntu2204 dbus-daemon[23409]:  [system] Activating via systemd: service name='org.freedesktop.hostname1' unit='dbus-org.freedesktop.hostname1.service' requested by ':1.3' (uid=100 pid=685 comm="/lib/systemd/systemd-networkd " label="unconfined")
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  enc1: DHCPv4 address 172.16.0.84/24 via 172.16.0.1
      Jun 20 22:20:07 ubuntu2204 systemd-timesyncd[23409]:  Network configuration changed, trying to establish connection.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Hostname Service...
      Jun 20 22:20:07 ubuntu2204 dbus-daemon[23409]:  [system] Successfully activated service 'org.freedesktop.hostname1'
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Hostname Service.
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  Could not set hostname: Access denied
      Jun 20 22:20:07 ubuntu2204 systemd-timesyncd[23409]:  Initial synchronization to time server 91.189.94.4:123 (ntp.ubuntu.com).
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  enc1: Gained IPv6LL
      Jun 20 22:20:07 ubuntu2204 systemd-networkd-wait-online[23409]:  managing: enc1
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Wait for Network to be Configured.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Network is Online.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Docker Application Container Engine...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Podman auto-update service...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Logging Configuration...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Condition check resulted in Ubuntu Pro Background Auto Attach being skipped.
      Jun 20 22:20:07 ubuntu2204 hpcr-dnslookup[23409]:  HPL14000I: Network connectivity check completed successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Logging Configuration.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Early Initialization.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Logging to remote monitoring server is initiated..
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Logging Configuration...
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  Configuring logging ...
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  Version [1.1.133]
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  Configuring logging, input [/var/hyperprotect/user-data.decrypted] ...
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  ValidateContractE ...
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.239817630Z" level=info msg="Starting up"
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.240238124Z" level=info msg="detected 127.0.0.53 nameserver, assuming systemd-resolved, so using resolv.conf: /run/systemd/resolve/resolv.conf"
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  config written: /etc/rsyslog.d/22-logging.conf
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  HPL01010I: Logging has been setup successfully.
      Jun 20 22:20:07 ubuntu2204 hpcr-logging[23409]:  Logging has been configured
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Logging Configuration.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting System Logging Service...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started System Logging Service.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Synchronizes the Logging Target.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Logging to remote log server is initiated..
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that does validation of contract...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting HPCR Registry Authentication...
      Jun 20 22:20:07 ubuntu2204 rsyslogd[23409]:  rsyslogd's groupid changed to 111
      Jun 20 22:20:07 ubuntu2204 rsyslogd[23409]:  rsyslogd's userid changed to 104
      Jun 20 22:20:07 ubuntu2204 rsyslogd[23409]:  [origin software="rsyslogd" swVersion="8.2112.0" x-pid="866" x-info="https://www.rsyslog.com"] start
      Jun 20 22:20:07 ubuntu2204 rsyslogd[23409]:  imjournal: No statefile exists, /var/spool/rsyslog/journal_state will be created (ignore if this is first run): No such file or directory [v8.2112.0 try https://www.rsyslog.com/e/2040 ]
      Jun 20 22:20:07 ubuntu2204 hpcr-contract[23409]:  Welcome to SE Contract Validator
      Jun 20 22:20:07 ubuntu2204 hpcr-contract[23409]:  Contract file passed is:  /var/hyperprotect/user-data.decrypted
      Jun 20 22:20:07 ubuntu2204 hpcr-registry-auth[23409]:  Starting Registry Authentication ...
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  audit: type=1400 audit(1687299605.285:9): apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=865 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 audit[23409]:  AVC apparmor="STATUS" operation="profile_load" profile="unconfined" name="docker-default" pid=865 comm="apparmor_parser"
      Jun 20 22:20:07 ubuntu2204 rsyslogd[23409]:  imjournal: journal files changed, reloading...  [v8.2112.0 try https://www.rsyslog.com/e/0 ]
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.296913492Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.296929973Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.296945131Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.296952791Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.298087429Z" level=info msg="parsed scheme: \"unix\"" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.298093999Z" level=info msg="scheme \"unix\" not registered, fallback to default scheme" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.298101688Z" level=info msg="ccResolverWrapper: sending update to cc: {[{unix:///run/containerd/containerd.sock  <nil> 0 <nil>}] <nil> <nil>}" module=grpc
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.298115182Z" level=info msg="ClientConn switching balancer to \"pick_first\"" module=grpc
      Jun 20 22:20:07 ubuntu2204 hpcr-registry-auth[23409]:  Version [1.0.61]
      Jun 20 22:20:07 ubuntu2204 hpcr-registry-auth[23409]:  Writing auth config: /root/.docker/config.json
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished HPCR Registry Authentication.
      Jun 20 22:20:07 ubuntu2204 hpcr-registry-auth[23409]:  Registry Authentication started
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.414520529Z" level=info msg="Loading containers: start."
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  bridge: filtering via arp/ip/ip6tables is no longer available by default. Update your scripts to load br_netfilter if you need this.
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Bridge firewalling registered
      Jun 20 22:20:07 ubuntu2204 hpcr-contract[23409]:  Contract file is valid.
      Jun 20 22:20:07 ubuntu2204 hpcr-contract[23409]:  Extracting workload from /var/hyperprotect/user-data.decrypted to /var/hyperprotect/workload-data.decrypted
      Jun 20 22:20:07 ubuntu2204 hpcr-contract[23409]:  Extraction completed
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Service that does validation of contract.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that does signature validation of Env Workload of contract...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  podman-auto-update.service: Deactivated successfully.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Podman auto-update service.
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Welcome to SE ENV Workload Signature Validator
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Decrypted Contract file passed is:  /var/hyperprotect/workload-data.decrypted
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Encrypted Contract file passed is:  /var/hyperprotect/cidata/user-data
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Check Dependency params Public key and EnvWorkload signature
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Access Public key and EnvWorkload signature
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Create combined EnvWorkload contract content
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Verify signing key, signature and combined EnvWorkload contract
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Verified OK
      Jun 20 22:20:07 ubuntu2204 hpcr-signature[23409]:  Successfully verified contract with signature and signing key
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Service that does signature validation of Env Workload of contract.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Contract is unpacked and ready for consumption..
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that waits until the user devices are ready...
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Set podman image policy...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  Waiting for devices ...
      Jun 20 22:20:07 ubuntu2204 kernel[23409]:  Initializing XFRM netlink socket
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  Version [1.0.87]
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.580722447Z" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  WaitForDevices input=[/var/hyperprotect/user-data.decrypted], timeout=[2023-06-20 22:35:05.580861902 +0000 UTC m=+900.020229758]
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  ParseContract ...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  ValidateContract ...
      Jun 20 22:20:07 ubuntu2204 systemd-udevd[23409]:  Using default interface naming scheme 'v249'.
      Jun 20 22:20:07 ubuntu2204 networkd-dispatcher[23409]:  WARNING:Unknown index 3 seen, reloading interface list
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  MergeVolumes ...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-standby[23409]:  Waiting for devices is completed
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Service that waits until the user devices are ready.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that mounts the data volumes after they are ready...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  Mounting volumes ...
      Jun 20 22:20:07 ubuntu2204 systemd-networkd[23409]:  docker0: Link UP
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  Version [1.0.87]
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  MountVolumes input=[/var/hyperprotect/user-data.decrypted]
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  ParseContract ...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  ValidateContract ...
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.638036029Z" level=info msg="Loading containers: done."
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  MergeVolumes ...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  Mounting volumes ...
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  Volume config ..
      Jun 20 22:20:07 ubuntu2204 hpcr-disk-mount[23409]:  HPL07003I: Mounting volumes done
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Service that mounts the data volumes after they are ready.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Reached target Data volumes are mounted ready to be used..
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Service that verifies all disks are encrypted and logs output to systemd journal.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Service that periodically logs entry to trigger verify disk encryption service.
      Jun 20 22:20:07 ubuntu2204 verify-disk-encryption[23409]:  Verify disk encryption started...
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Getting image source signatures
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Copying blob sha256:66d62867ae2452322f4769f943913be00b22e73039d1902e8f785b9f49838193
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.708061180Z" level=info msg="Docker daemon" commit="20.10.21-0ubuntu1~22.04.3" graphdriver(s)=overlay2 version=20.10.21
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.708111166Z" level=info msg="Daemon has completed initialization"
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Started Docker Application Container Engine.
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:05.759542231Z" level=info msg="API listen on /run/docker.sock"
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Copying config sha256:9eca761232387055827db0a9f2232f2635bc8c6d5f23ecfb39d34bb4ab0dca09
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Writing manifest to image destination
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Storing signatures
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Set docker image policy...
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Starting image service...
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Extracting image contract
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Successfully extracted Image contract
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Extracting container contract
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Checking for image with digest
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  No image for DCT verification
      Jun 20 22:20:07 ubuntu2204 hpcr-image[23409]:  Image service completed successfully
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Set docker image policy.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that creates a set of containers...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Starting container service...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Validating contract...
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Loaded image(s): k8s.gcr.io/pause:3.5
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Compose folder /data1/compose created
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Contract yaml file: /var/hyperprotect/workload-data.decrypted
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Compose folder: /data1/compose
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Validation completed
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Parsing contract...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Parsing of the Contract File completed successfully
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Extracting compose...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Extracting done...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Extracting the ENV Contents...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Writing new env file /data1/compose/.env ...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Reading existing env file /data1/compose/.env ...
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Extracting of environment contents done
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Check if docker is ready
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  docker-compose.yml file is present in the directory
      Jun 20 22:20:07 ubuntu2204 hpcr-container[23409]:  Starting workload containers...
      Jun 20 22:20:07 ubuntu2204 podman[23409]:  2023-06-20 22:20:05.692314364 +0000 UTC m=+0.134422634 image loadfromarchive  /usr/local/se-image-play/pause.tar
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-image-play
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  Version [1.1.94]
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:  pam_unix(sudo:session): session closed for user nobody
      Jun 20 22:20:07 ubuntu2204 hpcr-image-play[23409]:  tar: etc/containers/policy.json: time stamp 2023-06-20 22:20:06 is 0.099081804 s in the future
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Set podman image policy.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Starting Service that creates a set of containers...
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:      root : PWD=/ ; USER=nobody ; COMMAND=/usr/local/bin/se-container-play
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:  pam_unix(sudo:session): session opened for user nobody(uid=65534) by (uid=0)
      Jun 20 22:20:07 ubuntu2204 hpcr-container-play[23409]:  Version [1.1.99]
      Jun 20 22:20:07 ubuntu2204 sudo[23409]:  pam_unix(sudo:session): session closed for user nobody
      Jun 20 22:20:07 ubuntu2204 hpcr-container-play[23409]:  HPL15004I: The pod started successfully.
      Jun 20 22:20:07 ubuntu2204 hpcr-container-play[23409]:  HPL15006I: No pod definitions found.
      Jun 20 22:20:07 ubuntu2204 systemd[23409]:  Finished Service that creates a set of containers.
      Jun 20 22:20:07 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:06.057848531Z" level=warning msg="reference for unknown type: " digest="sha256:b0921f4009b33b926aeae931fef2b0536514e7a62ae013cee6c345b1ac7f11bb" remote="quay.io/bsilliman/paynow@sha256:b0921f4009b33b926aeae931fef2b0536514e7a62ae013cee6c345b1ac7f11bb"
      Jun 20 22:20:08 ubuntu2204 systemd[23409]:  dmesg.service: Deactivated successfully.
      Jun 20 22:20:08 ubuntu2204 systemd[23409]:  var-lib-containers-storage-overlay.mount: Deactivated successfully.
      Jun 20 22:20:08 ubuntu2204 systemd[23409]:  podman.service: Deactivated successfully.
      Jun 20 22:20:10 ubuntu2204 verify-disk-encryption[23409]:  HPL13000I: Verify LUKS Encryption
      Jun 20 22:20:10 ubuntu2204 systemd[23409]:  verify-disk-encryption-invoker.service: Deactivated successfully.
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value for disk-encrypt: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Executed cmd: ('lsblk', '-b', '-n', '-o', 'NAME,SIZE')
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Stdout: vda                                           107374182400
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  ├─vda1                                          4292870144
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  └─vda2                                        103079215104
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:    └─luks-f151eb69-04c7-42d5-8859-105dc2fd8512 103062437888
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  vdb                                                 389120
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  List of volumes greater than or equal to 10GB are: ['/dev/vda']
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Updated Volumes list: ['/dev/vda2']
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,MOUNTPOINT')
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Stdout: vda2
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  └─luks-f151eb69-04c7-42d5-8859-105dc2fd8512 /
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Boot volume is /dev/vda2
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Volume /dev/vda2 has mount point /
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  List of mounted volumes are: ['/dev/vda2']
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Verifying the boot disk /dev/vda2 is encrypted or not
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Executed cmd: ('lsblk', '/dev/vda2', '-b', '-n', '-o', 'NAME,TYPE')
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Stdout: vda2                                        part
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  └─luks-f151eb69-04c7-42d5-8859-105dc2fd8512 crypt
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Executed cmd: ('cryptsetup', 'isLuks', '/dev/vda2')
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Executed cmd: ('cryptsetup', 'luksDump', '/dev/vda2')
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Return value: 0
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  Checked for mount point /, LUKS encryption with 1 key slot found
      Jun 20 22:20:12 ubuntu2204 verify-disk-encryption[23409]:  HPL13001I: Boot volume and all the mounted data volumes are encrypted
      Jun 20 22:20:26 ubuntu2204 systemd-udevd[23409]:  Using default interface naming scheme 'v249'.
      Jun 20 22:20:26 ubuntu2204 networkd-dispatcher[23409]:  WARNING:Unknown index 4 seen, reloading interface list
      Jun 20 22:20:26 ubuntu2204 systemd-networkd[23409]:  br-4bdc8f6c9344: Link UP
      Jun 20 22:20:26 ubuntu2204 systemd[23409]:  var-lib-docker-overlay2-ad8ce6c1954354eb7a7b3649fabba5058df9ae11fc40eb562f85a0b37beefa6f\x2dinit-merged.mount: Deactivated successfully.
      Jun 20 22:20:28 ubuntu2204 networkd-dispatcher[23409]:  WARNING:Unknown index 5 seen, reloading interface list
      Jun 20 22:20:28 ubuntu2204 systemd-networkd[23409]:  veth414dfc5: Link UP
      Jun 20 22:20:28 ubuntu2204 systemd-udevd[23409]:  Using default interface naming scheme 'v249'.
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  br-4bdc8f6c9344: port 1(veth414dfc5) entered blocking state
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  br-4bdc8f6c9344: port 1(veth414dfc5) entered disabled state
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  device veth414dfc5 entered promiscuous mode
      Jun 20 22:20:28 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:27.899646942Z" level=info msg="No non-localhost DNS nameservers are left in resolv.conf. Using default external servers: [nameserver 8.8.8.8 nameserver 8.8.4.4]"
      Jun 20 22:20:28 ubuntu2204 dockerd[23409]:  time="2023-06-20T22:20:27.899671960Z" level=info msg="IPv6 enabled; Adding default IPv6 external servers: [nameserver 2001:4860:4860::8888 nameserver 2001:4860:4860::8844]"
      Jun 20 22:20:28 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:27.947464405Z" level=info msg="loading plugin \"io.containerd.event.v1.publisher\"..." runtime=io.containerd.runc.v2 type=io.containerd.event.v1
      Jun 20 22:20:28 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:27.947522915Z" level=info msg="loading plugin \"io.containerd.internal.v1.shutdown\"..." runtime=io.containerd.runc.v2 type=io.containerd.internal.v1
      Jun 20 22:20:28 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:27.947532571Z" level=info msg="loading plugin \"io.containerd.ttrpc.v1.task\"..." runtime=io.containerd.runc.v2 type=io.containerd.ttrpc.v1
      Jun 20 22:20:28 ubuntu2204 containerd[23409]:  time="2023-06-20T22:20:27.947665970Z" level=info msg="starting signal loop" namespace=moby path=/run/containerd/io.containerd.runtime.v2.task/moby/89f31672e75b906283166a7d8eb26f12bdd5e66949a93b4a47dfa30e025f6bc5 pid=1293 runtime=io.containerd.runc.v2
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Started libcontainer container 89f31672e75b906283166a7d8eb26f12bdd5e66949a93b4a47dfa30e025f6bc5.
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  eth0: renamed from vethf198c3e
      Jun 20 22:20:28 ubuntu2204 systemd-networkd[23409]:  veth414dfc5: Gained carrier
      Jun 20 22:20:28 ubuntu2204 systemd-networkd[23409]:  br-4bdc8f6c9344: Gained carrier
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  IPv6: ADDRCONF(NETDEV_CHANGE): veth414dfc5: link becomes ready
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  br-4bdc8f6c9344: port 1(veth414dfc5) entered blocking state
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  br-4bdc8f6c9344: port 1(veth414dfc5) entered forwarding state
      Jun 20 22:20:28 ubuntu2204 kernel[23409]:  IPv6: ADDRCONF(NETDEV_CHANGE): br-4bdc8f6c9344: link becomes ready
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Docker Compose Logs:
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:   paynow Pulling
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Pulling fs layer
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Waiting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Downloading [>                                                  ]  52.42kB/5.149MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Downloading [>                                                  ]  110.1kB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Downloading [===================================>               ]  7.628MB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [>                                                  ]  540.7kB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [========>                                          ]  9.111MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [==================>                                ]  19.86MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [>                                                  ]  527.5kB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [========================>                          ]  25.75MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [=====>                                             ]  5.863MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [===============================>                   ]  33.27MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [============>                                      ]  13.38MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [=====================================>             ]  40.29MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [===============>                                   ]   16.6MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [=========================================>         ]  44.05MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [======================>                            ]  24.12MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Downloading [============================================>      ]  47.84MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [=============================>                     ]  31.66MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [>                                                  ]  557.1kB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [===>                                               ]  3.342MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [==================================>                ]  37.02MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=====>                                             ]  6.128MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [>                                                  ]  539.8kB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [========================================>          ]  43.98MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [========>                                          ]  8.913MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Downloading [=================================================> ]  53.11MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [===========>                                       ]  12.26MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [==============>                                    ]   15.6MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [>                                                  ]  1.075MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Downloading [=======>                                           ]     605B/4.203kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Downloading [==================================================>]  4.203kB/4.203kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=================>                                 ]  18.94MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=>                                                 ]  4.278MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [====================>                              ]  22.28MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [==>                                                ]  9.631MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [>                                                  ]  474.2kB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=======================>                           ]  25.07MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [====>                                              ]  15.52MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [=======>                                           ]  6.609MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [======>                                            ]  22.47MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=========================>                         ]   27.3MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Downloading [>                                                  ]  23.27kB/2.277MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [============>                                      ]  11.78MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [================================>                  ]  34.54MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [===================>                               ]  17.88MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=====================================>             ]  39.55MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Downloading [===================>                               ]  898.8kB/2.277MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [========================>                          ]   22.6MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [========>                                          ]  27.85MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=======================================>           ]  42.34MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [============================>                      ]  26.38MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=========>                                         ]  33.73MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=========================================>         ]  44.56MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [============>                                      ]   42.3MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [===============>                                   ]  53.02MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [===============================>                   ]  29.21MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Downloading [==================================================>]     452B/452B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [==========================================>        ]  45.68MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=================>                                 ]  59.97MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [====================================>              ]  33.93MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [===========================================>       ]  46.79MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [===================>                               ]  66.93MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Downloading [============================================>      ]  40.54MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [==============================================>    ]  49.02MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=====================>                             ]  74.98MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Downloading [==================================================>]     126B/126B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [========================>                          ]  83.59MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [================================================>  ]  51.25MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [===========================>                       ]   93.8MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [==============================>                    ]  104.5MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [=================================================> ]  52.36MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Downloading [====>                                              ]     612B/6.827kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Downloading [==================================================>]  6.827kB/6.827kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=================================>                 ]  115.3MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Extracting [==================================================>]  53.28MB/53.28MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Downloading [>                                                  ]   21.9kB/2.143MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [===================================>               ]  123.8MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Downloading [==================================>                ]  1.484MB/2.143MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [======================================>            ]  134.6MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Downloading [>                                                  ]  10.14kB/902.5kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=========================================>         ]  143.2MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [============================================>      ]  152.9MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [===============================================>   ]  162.5MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Downloading [=================================================> ]  172.2MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Verifying Checksum
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Download complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  51f6de0debe6 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Extracting [>                                                  ]  65.54kB/5.149MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Extracting [===================================>               ]   3.67MB/5.149MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Extracting [==================================================>]  5.149MB/5.149MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  6404e912b557 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Extracting [>                                                  ]  131.1kB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Extracting [================>                                  ]  3.539MB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Extracting [=================================================> ]  10.62MB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Extracting [==================================================>]  10.77MB/10.77MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0561ee66ff6a Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [>                                                  ]  557.1kB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [===>                                               ]  3.899MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [======>                                            ]  7.242MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=========>                                         ]  10.58MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [============>                                      ]  13.93MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [===============>                                   ]  17.27MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [==================>                                ]  20.05MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=====================>                             ]  22.84MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=======================>                           ]  25.62MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [==========================>                        ]  28.41MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [===========================>                       ]  30.08MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=============================>                     ]  32.31MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [================================>                  ]  35.09MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [===================================>               ]  38.44MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [======================================>            ]  41.78MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=========================================>         ]  45.12MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [============================================>      ]  48.46MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [===============================================>   ]  51.25MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [=================================================> ]  53.48MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Extracting [==================================================>]  54.06MB/54.06MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7a6c7ccf7cb5 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [>                                                  ]  557.1kB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=>                                                 ]  3.899MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==>                                                ]  7.799MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===>                                               ]   11.7MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [====>                                              ]  13.93MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [====>                                              ]  15.04MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [====>                                              ]  16.71MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=====>                                             ]  18.38MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=====>                                             ]   19.5MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [======>                                            ]  21.73MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=======>                                           ]  25.07MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [========>                                          ]  28.97MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=========>                                         ]  32.31MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==========>                                        ]  36.21MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===========>                                       ]  39.55MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [============>                                      ]  42.34MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=============>                                     ]  45.68MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==============>                                    ]  49.58MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==============>                                    ]  51.81MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===============>                                   ]  54.59MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [================>                                  ]  57.93MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================>                                 ]  61.28MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==================>                                ]  64.62MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===================>                               ]  67.96MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [====================>                              ]   71.3MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=====================>                             ]  74.65MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [======================>                            ]  77.99MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=======================>                           ]  81.33MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [========================>                          ]  85.23MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=========================>                         ]  89.13MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==========================>                        ]  92.47MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===========================>                       ]  96.37MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=============================>                     ]  100.3MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=============================>                     ]  103.6MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==============================>                    ]    107MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===============================>                   ]  110.3MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [================================>                  ]  113.6MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================================>                 ]  114.8MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================================>                 ]  116.4MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==================================>                ]  119.2MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===================================>               ]  123.1MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [====================================>              ]  126.5MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=====================================>             ]  129.2MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=======================================>           ]  135.9MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=========================================>         ]  144.8MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [============================================>      ]  152.6MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=============================================>     ]  157.1MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==============================================>    ]  159.9MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===============================================>   ]  162.7MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [===============================================>   ]  165.4MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [================================================>  ]  168.2MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================================================> ]  170.5MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================================================> ]  171.6MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [=================================================> ]  172.7MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Extracting [==================================================>]  172.8MB/172.8MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  2854c5c8fd87 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Extracting [==================================================>]  4.203kB/4.203kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Extracting [==================================================>]  4.203kB/4.203kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  65a3fb6c13d7 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [>                                                  ]  491.5kB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [====>                                              ]  3.932MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [========>                                          ]  7.373MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===========>                                       ]  10.81MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===============>                                   ]  14.25MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===================>                               ]  17.69MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [=======================>                           ]  21.63MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===========================>                       ]  25.56MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===============================>                   ]     29MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===================================>               ]  32.44MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [=====================================>             ]   34.9MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [========================================>          ]  36.86MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [==========================================>        ]  38.83MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [============================================>      ]   40.8MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [=============================================>     ]  41.78MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [===============================================>   ]  43.75MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [================================================>  ]  44.73MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [=================================================> ]  45.71MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Extracting [==================================================>]  46.04MB/46.04MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  c9fe958b3ae4 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Extracting [>                                                  ]  32.77kB/2.277MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Extracting [======================================>            ]  1.737MB/2.277MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Extracting [==================================================>]  2.277MB/2.277MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  5c31cf8345fa Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Extracting [==================================================>]     452B/452B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Extracting [==================================================>]     452B/452B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  7b32651b0169 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Extracting [==================================================>]     126B/126B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Extracting [==================================================>]     126B/126B
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  0d117aef64cc Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Extracting [==================================================>]  6.827kB/6.827kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Extracting [==================================================>]  6.827kB/6.827kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  197acf2cade1 Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Extracting [>                                                  ]  32.77kB/2.143MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Extracting [===============>                                   ]  655.4kB/2.143MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Extracting [==================================================>]  2.143MB/2.143MB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  d78e4d77283b Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Extracting [=>                                                 ]  32.77kB/902.5kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Extracting [==================================================>]  902.5kB/902.5kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Extracting [==================================================>]  902.5kB/902.5kB
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  8b3975218acc Pull complete
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  paynow Pulled
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Network compose_default  Creating
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Network compose_default  Created
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Container compose-paynow-1  Creating
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Container compose-paynow-1  Created
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Container compose-paynow-1  Starting
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Container compose-paynow-1  Started
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Docker compose result:
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:   CONTAINER ID   IMAGE                      COMMAND                  CREATED         STATUS                  PORTS                                       NAMES
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  89f31672e75b   quay.io/bsilliman/paynow   "docker-entrypoint.s…"   2 seconds ago   Up Less than a second   0.0.0.0:8443->8443/tcp, :::8443->8443/tcp   compose-paynow-1
      Jun 20 22:20:28 ubuntu2204 hpcr-container[23409]:  Container service completed successfully
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Finished Service that creates a set of containers.
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Reached target Workload is up and running..
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Starting Phase2 Catch Service...
      Jun 20 22:20:28 ubuntu2204 hpcr-catch-success[23409]:  VSI has started successfully.
      Jun 20 22:20:28 ubuntu2204 hpcr-catch-success[23409]:  HPL10001I: Services succeeded -> systemd triggered hpl-catch-success service
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Finished Phase2 Catch Service.
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Reached target Multi-User System.
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Starting Record Runlevel Change in UTMP...
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  systemd-update-utmp-runlevel.service: Deactivated successfully.
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Finished Record Runlevel Change in UTMP.
      Jun 20 22:20:28 ubuntu2204 systemd[23409]:  Startup finished in 26.715s (kernel) + 25.987s (userspace) = 52.702s.
      Jun 20 22:20:28 ubuntu2204 compose-paynow-1[23409]:  
      Jun 20 22:20:28 ubuntu2204 compose-paynow-1[23409]:  > hyper-protect-pay-now@1.0.0 start
      Jun 20 22:20:28 ubuntu2204 compose-paynow-1[23409]:  > node app.js
      Jun 20 22:20:28 ubuntu2204 compose-paynow-1[23409]:  
      Jun 20 22:20:29 ubuntu2204 systemd-networkd[23409]:  veth414dfc5: Gained IPv6LL
      Jun 20 22:20:30 ubuntu2204 systemd-networkd[23409]:  br-4bdc8f6c9344: Gained IPv6LL
      ```

Please click the *Next* link at the bottom right of the page to continue with the lab.
