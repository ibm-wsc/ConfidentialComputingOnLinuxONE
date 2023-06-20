# Try to find sensitive data in core dump


## Switch to your RHEL host terminal session

Switch to your terminal tab or window for your RHEL host session:

<img src="../../../images/RHELHost.png" width="351" height="217" />

Switch to your home directory:

   ``` bash
   cd ${HOME} 
   ```

Take a core dump of your HPVS KVM guest running the PayNow demo:

   ``` bash
   sudo /usr/local/bin/gcoreMyPaynowGuest.sh
   ```

???- example "Sample output from dumping your HPVS KVM guest"

      ```
      ```

Optional: Display the contents of the script you just ran if you are curious as to how it worked:

   ``` bash
   cat /usr/local/bin/gcoreMyPaynowGuest.sh
   ```

???- example "Contents of the gcoreMyPaynowGuest.sh script"

      ```
      my PayNow HPVS guest Pid is 2122519
      [New LWP 2122558]
      [New LWP 2122563]
      [New LWP 2122564]
      [New LWP 2122565]
      [New LWP 2124052]
      [New LWP 2124054]
      [New LWP 2124056]
      [New LWP 2124057]
      [Thread debugging using libthread_db enabled]
      Using host libthread_db library "/lib64/libthread_db.so.1".
      0x000003ffb3cf11f4 in ppoll () from target:/lib64/libc.so.6
      Saved corefile core.2122519
      [Inferior 1 (process 2122519) detached]
      ```

The script runs with root authority-  it lists processes, grabs the process ID for your HPVS guest running PayNow, takes a core (memory) dump of the process, and then assigns your userid ownership of the dump file.


Set an environment variable for the process ID for your Ubuntu KVM guest.  The script you ran did this as well but it was only set for the duration of the script execution, so you need to do it again:

   ``` bash
   myHPVSPaynowPid=$(ps aux | grep qemu | grep paynowse$(temp=$(whoami) && echo ${temp: -2}) | awk '{print $2}')
   echo My HPVS Guest for PayNow demo process id is ${myHPVSPaynowPid}
   ```

Attempt to pick out sensitive credit information from the core dump:

   ``` bash
   strings core.${myHPVSPaynowPid} | grep creditCard
   ```

In contrast to what you saw when performing the above procedure against a standard KVM guest that was running the PayNow demo app, this time you do not see any sensitive data, due to the protection offered by Hyper Protect Virtual Servers 2.1.5!! You are protected from malicious insider attacks!


Please click the *Next* button at the lower right of the page in order to perform lab cleanup.

