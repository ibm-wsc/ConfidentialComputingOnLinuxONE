# Demonstrate the protection of the Secure Execution-enabled HPVS 2.1.3 guest

## Overview of this section

In this section you will demonstrate the protection offered by the Secure Execution-enabled HPVS 2.1.3 guest in contrast to the ease in which a malicious insider can eavesdrop on a standard KVM guest.

## Log out of your Ubuntu KVM guest 

All of the work in this section is performed on the RHEL 8.5 host, so log out of your Ubuntu KVM guest:

   ``` bash
   exit
   ```

## Log in to the RHEL 8.5 host:

   ``` bash
   ssh -l ${StudentID} 192.168.22.64
   ```

## Snoop into your standard KVM guest with ease

A systems administrator at the host level does not have a difficult time getting into a standard KVM guest's business.  Try this command to dump the entire address space of your Ubuntu KVM guest:

   ``` bash
   sudo virsh dump $(whoami) $(whoami).dump
   ```

This will take a few seconds but you have just dumped the entire memory of your KVM guest.

Look at the file size:

   ``` bash
   ls -lh $(whoami).dump
   ```

We suspect that a malicious actor might have a few more tools in their toolbag than what we will show you here, but try this command:

   ``` bash
   sudo strings $(whoami).dump
   ```

The above command will print out all of the strings it recognizes in the memory dump.  You are probably getting tired of seeing them pass by on your terminal screen, so type `Ctrl-C` when you want your command prompt back.

Try this command to see how many strings were found in the file:

   ``` bash
   sudo strings $(whoami).dump | wc --lines
   ```

Your output may differ, but when we tried this command while writing up the lab, we had 2,397,409 strings found in our dump.  Now we didn't dig much deeper than this, but it's possible that a motivated hacker might find something among those millions of strings with which to make mischief.

## Go ahead and try to hack me says the HPVS 2.1.3 guest

Try to snoop on your Secure Execution-enabled Hyper Protect Virtual Servers 2.1.3 guest that is running your GREP11 Server. See what happens:

   ``` bash
   suffix=$(temp=$(whoami) && echo ${temp: -2}) ; sudo virsh dump grep11se${suffix} grep11se${suffix}.dump
   ```

??? example "Shot down in flames, ain't it a shame"

    ```
    error: Failed to core dump domain 'grep11se01' to grep11se01.dump
    error: internal error: unable to execute QEMU command 'migrate': protected VMs are currently not migrateable.
    ```

The verdict is in:  protection of data in use is **good**.  Unprotected data in use is **not so good**.

Please proceed to the next section of the lab for lab cleanup.

