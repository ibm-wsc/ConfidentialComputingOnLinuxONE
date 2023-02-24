# Clean up the resources you created during the lab

All of the work in this section is performed on the RHEL 8.5 host.  You should already be logged in to it if you have been following this lab in order.

## Shut down your regular Ubuntu KVM guest

Enter this command to shut down your regular Ubuntu KVM guest:

   ``` bash
   sudo virsh shutdown $(whoami)
   ```

## Shut down your HPVS 2.1.3 guest (your GREP11 server):

   ``` bash
   suffix=$(temp=$(whoami) && echo ${temp: -2}) ; sudo virsh shutdown grep11se${suffix} 
   ```

## Clean up the home directory of your userid on the REHL 8.5 host:

   ``` bash 
   cd ${HOME} && rm -rf rsyslogWork GREP11CAwork contract $(whoami).dump
   ```

Thank you for cleaning up and congratulations on finishing the lab!  We hope you enjoyed it and learned from it and we welcome your feedback on how to make it better!

There is no need to click the `Next` link at the bottom as that will take you to a page that is for instructor usage.  Feel free to check it out though, as it will give you insight into the tools that we use to create and update the lab. 

