# Run OCI image

You will be performing this section from your Ubuntu KVM guest. 

Your window or tab should like like this (unless you customized the profile we provided you):

<img src="../../../images/KVMGuest.png" width="351" height="217" />

## List your OCI image

Run this command to see the OCI image that you created, as well as an image (_node:19_) that was pulled down and used as the _base_ image of your _paynow:latest_ image:

   ``` bash
   docker images
   ```

Your output will look like this: 

???+ example "Example output"

      ```
      REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
      paynow       latest    429e779b94ad   12 seconds ago   935MB
      node         19        7ecdfdb7699d   6 days ago       927MB
      ```

Run the OCI image:

   ``` bash
   docker run -it -p 8443:8443 paynow
   ```

You will see output like this:

???- example "Output from starting PayNow app"

      ```
      
      > hyper-protect-pay-now@1.0.0 start
      > node app.js
      
      ```

Please click *Next* on the bottom right of the page to continue.
