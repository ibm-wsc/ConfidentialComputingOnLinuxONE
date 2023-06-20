# Find sensitive data in core dump


## Switch to your RHEL host terminal session

Switch to your terminal tab or window for your RHEL host session:

<img src="../../../images/RHELHost.png" width="351" height="217" />

Take a core dump of your KVM guest:

   ``` bash
   sudo /usr/local/bin/gcoreMyGuest.sh
   ```

???- example "Sample output from dumping your KVM guest"

      ```
      ```

Optional: Display the contents of the script you just ran if you are curious as to how it worked:

   ``` bash
   cat /usr/local/bin/gcoreMyGuest.sh
   ```

???- example "Contents of the gcoreMyGuest.sh script"

      ```
      #!/bin/sh
      
      myPid=$(ps aux | grep qemu | grep guest=${SUDO_USER} | awk '{print $2}')
      
      echo my guest Pid is ${myPid}
      
      /opt/rh/gcc-toolset-9/root/usr/bin/gcore ${myPid} 2>/dev/null
      
      chown ${SUDO_USER}:hpvs_students /home/${SUDO_USER}/core.${myPid}
      ```

The script runs with root authority-  it lists processes, grabs the process ID for your Ubuntu KVM guest, takes a core (memory) dump of the process, and then assigns your userid ownership of the dump file.


Set an environment variable for the process ID for your Ubuntu KVM guest.  The script you ran did this as well but it was only set for the duration of the script execution, so you need to do it again:

   ``` bash
   myPid=$(ps aux | grep qemu | grep $(whoami) | awk '{print $2}')
   echo My Ubuntu KVM Guest process id is ${myPid}
   ```

Pick out sensitive credit information from the core dump:

   ``` bash
   strings core.${myPid} | grep creditCard
   ```

You should recognize the sensitve information that you entered in the PayNow demo app.  This demonstrates how a malicious system administrator on the KVM host can look at sensitive information from a standard KVM guest.  An HPVS 2.1.5 KVM guest, which is protected by Secure Execution, prevents this from occurring, as you will see from completing the remainder of this lab.

Please click *Next* below to continue
