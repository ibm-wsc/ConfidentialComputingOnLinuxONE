# Create OCI Image
                
You will be performing this section from your Ubuntu KVM guest. 

Your window or tab should like like this (unless you customized the profile we provided you):
        
<img src="../../../images/KVMGuest.png" width="351" height="217" />

## Install Docker

You will need to install Docker on your guest.  You can see that it is not currently installed by running the following command:

   ``` bash
   which docker || echo 'Docker is not found'
   ```

???- Example "Output showing Docker is not found"

      ```
      Docker is not found
      ```

See what version of Docker is available to install with this command:

   ``` bash
   sudo apt-cache policy docker.io
   ```

You can see from the output that _docker.io_ is not currently installed and that version _20.10.21_ is the candidate, or suggested, version to install:

???- Example "Output showing Docker version to install"

      ```
      docker.io:
        Installed: (none)
        Candidate: 20.10.21-0ubuntu1~22.04.2
        Version table:
           20.10.21-0ubuntu1~22.04.2 500
              500 http://ports.ubuntu.com/ubuntu-ports jammy-updates/universe s390x Packages
           20.10.12-0ubuntu4 500
              500 http://ports.ubuntu.com/ubuntu-ports jammy/universe s390x Packages
      ```

Install Docker with this command:

   ``` bash
   sudo apt install docker.io
   ```

Type **Y** and press *Return* when asked if you want to continue with the installation.

???- Example "Output from Docker installation"

      ```
      Reading package lists... Done
      Building dependency tree... Done
      Reading state information... Done
      The following packages were automatically installed and are no longer required:
        linux-headers-5.15.0-60 linux-headers-5.15.0-60-generic linux-image-5.15.0-60-generic linux-modules-5.15.0-60-generic
        linux-modules-extra-5.15.0-60-generic
      Use 'sudo apt autoremove' to remove them.
      The following additional packages will be installed:
        bridge-utils containerd dns-root-data dnsmasq-base pigz runc ubuntu-fan
      Suggested packages:
        ifupdown aufs-tools cgroupfs-mount | cgroup-lite debootstrap docker-doc rinse zfs-fuse | zfsutils
      The following NEW packages will be installed:
        bridge-utils containerd dns-root-data dnsmasq-base docker.io pigz runc ubuntu-fan
      0 upgraded, 8 newly installed, 0 to remove and 0 not upgraded.
      Need to get 56.2 MB of archives.
      After this operation, 251 MB of additional disk space will be used.
      Do you want to continue? [Y/n] Y
      Get:1 http://ports.ubuntu.com/ubuntu-ports jammy/universe s390x pigz s390x 2.6-1 [67.2 kB]
      Get:2 http://ports.ubuntu.com/ubuntu-ports jammy/main s390x bridge-utils s390x 1.7-1ubuntu3 [34.3 kB]
      Get:3 http://ports.ubuntu.com/ubuntu-ports jammy-updates/main s390x runc s390x 1.1.4-0ubuntu1~22.04.1 [4115 kB]
      Get:4 http://ports.ubuntu.com/ubuntu-ports jammy-updates/main s390x containerd s390x 1.6.12-0ubuntu1~22.04.1 [26.3 MB]
      Get:5 http://ports.ubuntu.com/ubuntu-ports jammy/main s390x dns-root-data all 2021011101 [5256 B]
      Get:6 http://ports.ubuntu.com/ubuntu-ports jammy-updates/main s390x dnsmasq-base s390x 2.86-1.1ubuntu0.2 [347 kB]
      Get:7 http://ports.ubuntu.com/ubuntu-ports jammy-updates/universe s390x docker.io s390x 20.10.21-0ubuntu1~22.04.2 [25.3 MB]
      Get:8 http://ports.ubuntu.com/ubuntu-ports jammy/universe s390x ubuntu-fan all 0.12.16 [35.2 kB]
      Fetched 56.2 MB in 2s (22.6 MB/s)     
      Preconfiguring packages ...
      Selecting previously unselected package pigz.
      (Reading database ... 101325 files and directories currently installed.)
      Preparing to unpack .../0-pigz_2.6-1_s390x.deb ...
      Unpacking pigz (2.6-1) ...
      Selecting previously unselected package bridge-utils.
      Preparing to unpack .../1-bridge-utils_1.7-1ubuntu3_s390x.deb ...
      Unpacking bridge-utils (1.7-1ubuntu3) ...
      Selecting previously unselected package runc.
      Preparing to unpack .../2-runc_1.1.4-0ubuntu1~22.04.1_s390x.deb ...
      Unpacking runc (1.1.4-0ubuntu1~22.04.1) ...
      Selecting previously unselected package containerd.
      Preparing to unpack .../3-containerd_1.6.12-0ubuntu1~22.04.1_s390x.deb ...
      Unpacking containerd (1.6.12-0ubuntu1~22.04.1) ...
      Selecting previously unselected package dns-root-data.
      Preparing to unpack .../4-dns-root-data_2021011101_all.deb ...
      Unpacking dns-root-data (2021011101) ...
      Selecting previously unselected package dnsmasq-base.
      Preparing to unpack .../5-dnsmasq-base_2.86-1.1ubuntu0.2_s390x.deb ...
      Unpacking dnsmasq-base (2.86-1.1ubuntu0.2) ...
      Selecting previously unselected package docker.io.
      Preparing to unpack .../6-docker.io_20.10.21-0ubuntu1~22.04.2_s390x.deb ...
      Unpacking docker.io (20.10.21-0ubuntu1~22.04.2) ...
      Selecting previously unselected package ubuntu-fan.
      Preparing to unpack .../7-ubuntu-fan_0.12.16_all.deb ...
      Unpacking ubuntu-fan (0.12.16) ...
      Setting up dnsmasq-base (2.86-1.1ubuntu0.2) ...
      Setting up runc (1.1.4-0ubuntu1~22.04.1) ...
      Setting up dns-root-data (2021011101) ...
      Setting up bridge-utils (1.7-1ubuntu3) ...
      Setting up pigz (2.6-1) ...
      Setting up containerd (1.6.12-0ubuntu1~22.04.1) ...
      Created symlink /etc/systemd/system/multi-user.target.wants/containerd.service → /lib/systemd/system/containerd.service.
      Setting up ubuntu-fan (0.12.16) ...
      Created symlink /etc/systemd/system/multi-user.target.wants/ubuntu-fan.service → /lib/systemd/system/ubuntu-fan.service.
      Setting up docker.io (20.10.21-0ubuntu1~22.04.2) ...
      Adding group `docker' (GID 121) ...
      Done.
      Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /lib/systemd/system/docker.service.
      Created symlink /etc/systemd/system/sockets.target.wants/docker.socket → /lib/systemd/system/docker.socket.
      Processing triggers for dbus (1.12.20-2ubuntu4.1) ...
      Processing triggers for man-db (2.10.2-1) ...
      Scanning processes...                                                                                                                    
      Scanning linux images...                                                                                                                 
      
      Running kernel seems to be up-to-date (ABI upgrades are not detected).
      
      No services need to be restarted.
      
      No containers need to be restarted.
      
      No user sessions are running outdated binaries.
      
      No VM guests are running outdated hypervisor (qemu) binaries on this host.
      ```

Repeat this command from earlier and you'll now see that Docker is installed:

   ``` bash
   sudo apt-cache policy docker.io
   ```

???- Example "Output showing docker.io is installed"

      ```
      docker.io:
        Installed: 20.10.21-0ubuntu1~22.04.2
        Candidate: 20.10.21-0ubuntu1~22.04.2
        Version table:
       *** 20.10.21-0ubuntu1~22.04.2 500
              500 http://ports.ubuntu.com/ubuntu-ports jammy-updates/universe s390x Packages
              100 /var/lib/dpkg/status
           20.10.12-0ubuntu4 500
              500 http://ports.ubuntu.com/ubuntu-ports jammy/universe s390x Packages
      ```

Run this command to see where the _docker_ binary lives:

   ``` bash
   which docker && echo Docker is found!
   ```

???- Example "Output showing where the docker binary resides"

      ```
      /usr/bin/docker
      Docker is found!
      ```

Run this command to display the Docker version:

   ``` bash
   docker version
   ```

Besides noting the version, note the permission error at the bottom of the output:

???- Example "Docker version info, plus a permission error"

      ```
      Client:
       Version:           20.10.21
       API version:       1.41
       Go version:        go1.18.1
       Git commit:        20.10.21-0ubuntu1~22.04.2
       Built:             Thu Mar  2 18:26:19 2023
       OS/Arch:           linux/s390x
       Context:           default
       Experimental:      true
      Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version": dial unix /var/run/docker.sock: connect: permission denied
      ```


You need to add your userid to the _docker_ group in order to have permission to use Docker:

   ``` bash
   sudo usermod -aG docker student
   ```

(There is no output from the above command when it works).

You will need to log out and then log back in in order for your updated permissions to take effect.

Log out:

   ``` bash
   exit
   ```

Log back in:

   ``` bash
   ssh -p ${Student_SSH_Port} -l student 192.168.22.64
   ```

Now repeat _docker version_ and you should not see any errors and you should see more information as well:

   ``` bash
   docker version
   ```

???- Example "Output when your permissions are correct"

      ```
      Client:
       Version:           20.10.21
       API version:       1.41
       Go version:        go1.18.1
       Git commit:        20.10.21-0ubuntu1~22.04.2
       Built:             Thu Mar  2 18:26:19 2023
       OS/Arch:           linux/s390x
       Context:           default
       Experimental:      true
      
      Server:
       Engine:
        Version:          20.10.21
        API version:      1.41 (minimum version 1.12)
        Go version:       go1.18.1
        Git commit:       20.10.21-0ubuntu1~22.04.2
        Built:            Wed Feb 15 15:26:57 2023
        OS/Arch:          linux/s390x
        Experimental:     false
       containerd:
        Version:          1.6.12-0ubuntu1~22.04.1
        GitCommit:        
       runc:
        Version:          1.1.4-0ubuntu1~22.04.1
        GitCommit:        
       docker-init:
        Version:          0.19.0
        GitCommit: 
      ```

## Build OCI Image for PayNow demo

Switch to the proper directory:

   ``` bash
   cd ~/paynow-website && pwd
   ```


Before you get started, run this command to see that you currently have no OCI images on your system:

   ``` bash
   docker images
   ```

???- example "Expected output"

      ```
      REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
      ```

Now, build your OCI image containing the PayNow Demo.

   ``` bash
   docker build -t paynow .
   ```

You can ignore any warning messages. Example output is shown below.

???- example "Example output"

      ```
      Sending build context to Docker daemon  2.831MB
      Step 1/7 : FROM node:19
      19: Pulling from library/node
      042263756d7d: Pull complete 
      2973b8c98f9d: Pull complete 
      6f740d13ac61: Pull complete 
      09c0067783f4: Pull complete 
      ff1b5accdd9f: Pull complete 
      c857be20e701: Pull complete 
      84ecc9c3008a: Pull complete 
      b5252f19c11d: Pull complete 
      1cda3fcd02f6: Pull complete 
      Digest: sha256:b913dbec87493fb38077268785a48ab9b28c5454d0b03ca4dadc13929baf318e
      Status: Downloaded newer image for node:19
       ---> 7ecdfdb7699d
      Step 2/7 : WORKDIR /app
       ---> Running in f5eae6d94758
      Removing intermediate container f5eae6d94758
       ---> c07a591568f5
      Step 3/7 : COPY app/package*.json ./
       ---> 4fd0bb503e7b
      Step 4/7 : RUN npm install
       ---> Running in fa6592183ac7
      npm WARN old lockfile 
      npm WARN old lockfile The package-lock.json file was created with an old version of npm,
      npm WARN old lockfile so supplemental metadata must be fetched from the registry.
      npm WARN old lockfile 
      npm WARN old lockfile This is a one-time fix-up, please be patient...
      npm WARN old lockfile 
      
      added 64 packages, and audited 65 packages in 2s
      
      7 packages are looking for funding
        run `npm fund` for details
      
      found 0 vulnerabilities
      npm notice 
      npm notice New patch version of npm available! 9.6.3 -> 9.6.4
      npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.6.4>
      npm notice Run `npm install -g npm@9.6.4` to update!
      npm notice 
      Removing intermediate container fa6592183ac7
       ---> f7cec2decdab
      Step 5/7 : COPY app/ .
       ---> 0a6ad4a1751d
      Step 6/7 : EXPOSE 8443
       ---> Running in 9cf98c8e3a34
      Removing intermediate container 9cf98c8e3a34
       ---> 8f11e6dc8a71
      Step 7/7 : CMD npm start
       ---> Running in 52b41bb5df2e
      Removing intermediate container 52b41bb5df2e
       ---> 3942657125cc
      Successfully built 3942657125cc
      Successfully tagged paynow:latest
      ```

Please click the *Next* link at the lower right to continue in the lab.
