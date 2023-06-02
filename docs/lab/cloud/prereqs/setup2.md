# Set up environment variables used throughout the four IBM Cloud-based labs

On your prep system, there are three more environment variables that will be used in the four labs.

| Variable name | Description |
|---|---|
| LAB_WORKDIR | working directory for contract preparation |
| LOG_INGESTION_KEY | The ingestion key for your IBM Log Analysis instance |
| LOG_HOSTNAME | The hostname on IBM Cloud that hosts your IBM Log Analysis instance |

!!! Note

    These variables are set for the duration of your terminal session.  If you do not finish the labs in one terminal session, then you will need to revisit this section to set these variables again when you resume.

## Set LAB_WORKDIR

In order to avoid interference with other work, we want you to create a brand new directory for these labs- each of the four labs will use a subdirectory underneath the new directory.

1. This next instruction sets the environment variable to ${HOME}/cloudlabs.

    ``` bash
    LAB_WORKDIR=${HOME}/cloudlabs
    ```


2. This next set of commands will either successfully create this new directory and then change to it, or it will warn you to try again.  (If you are warned, retry step 1 above with a different name and then try these commands again.)

    ``` bash
    if [[ -e ${LAB_WORKDIR} ]]; then
       echo ${LAB_WORKDIR} already exists
       echo "  This may be appropriate if you are"
       echo "  resuming these labs in a new terminal session"
       echo "  and have already created this directory structure"
       echo
       echo Otherwise, choose a new value for LAB_WORKDIR
       echo "  or rename ${LAB_WORKDIR}"
       echo "  and try again"
    else
        mkdir ${LAB_WORKDIR} && \
        echo Fresh lab working directory created \
        && cd ${LAB_WORKDIR} && \
        echo Changed to working directory ${LAB_WORKDIR}
    fi
    ```

## Set LOG_INGESTION_KEY

1. Set an environment variable for your IBM Log Analysis ingestion key. You saved this somewhere safe earlier in the lab.  If you lost track of it, revisit the section [Create an IBM Log Analysis instance](../ibmlog){target="_blank" rel="noopener"} for the steps to retrieve it.

    ``` bash
    LOG_INGESTION_KEY="paste your ingestion key here"
    ```
       
## Set LOG_HOSTNAME

1. Set an environment variable for the hostname of your IBM Log Analysis instance. You saved this somewhere safe earlier in the lab. If you lost track of it, revisit the section [Create an IBM Log Analysis instance](../ibmlog){target="_blank" rel="noopener"} for the steps to retrieve it.
    
    ``` bash
    LOG_HOSTNAME="paste your IBM Log Analysis hostname here"
    ``` 
      
Click the **Next** link in the lower right to begin the first lab.

