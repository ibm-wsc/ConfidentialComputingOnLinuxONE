# (Optional) Create a server to use to create contracts

If you already have access to a Linux system or to a MacOS system you may skip this step.  If you have a Windows system and have the expertise to translate these instructions into something workable on Windows, you may skip this step.  

The lab authors have tested the instructions on Linux and macOS.

If you do not have a suitable system already or would like to, you can create a Hyper Protect Virtual Servers _Classic_ instance for free that will provide an Ubuntu system for you.  You will need to provide the public key portion of an SSH key pair when you provision this instance. 

**NOTE: Whether you already have a suitable system or choose to create one with the subsequent directions below, we will refer to this system as *your prep system* in the lab instructions, since you will use this system to *prepare* your contracts.**

To create this instance: 

1. Click on the **Catalog** link at the top of the page.

2. Start typing _Hyper Protect Virtual Servers_ in the search box until **Hyper Protect Virtual Servers for Classic** appears in the search results and then select it.

3. Choose the **Lite** plan- select the *region* and *zone* you prefer, we prefer to use the same *zone* that we used for our virtual private cloud.  (This instance will not be running inside a VPC)

4. Provide your SSH public key.

If you have created a *Hyper Protect Virtual Server for Classic* instance to use as your prep system, you need to install two packages:

1. Log in to your instance via ssh.

2. Enter the following command:

    ``` bash
    apt install -y curl vim
    ```
