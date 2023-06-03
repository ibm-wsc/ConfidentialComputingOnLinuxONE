# Prepare the contract 

## Make a new directory structure for the labs

These labs assume that ${HOME}/cloudlabs does not exist on your prep system. 

1. This command will warn you if this directory already exists:

    ``` bash
    mkdir ${HOME}/cloudlabs || echo '${HOME}/cloudlabs exists. Please use a new directory.'
    ```

2. This command will warn you if this directory already exists:

    ``` bash
    if [[ -e ${LAB_WORKDIR} ]]; then
       echo ${LAB_WORKDIR} already exists
       echo Please choose new value for LAB_WORKDIR
       echo '  or rename ${LAB_WORKDIR}
       echo '  and try again'
    else 
        mkdir ${LAB_WORKDIR} && \
        echo 'Fresh lab working directory created' \
        && cd ${LAB_WORKDIR} && \
        echo ' Changed to working directory ${LAB_WORKDIR}
    fi
    ```

1. Go to a command prompt on your prep system

1. Make a fresh directory structure and change in to it:

    ``` bash
    mkdir -p ${HOME}/cloudlabs/lab1 && cd ${HOME}/cloudlabs/lab1
    ```

1. Create another directory to hold a docker compose file and switch to it:

    ``` bash
    mkdir compose && cd compose
    ```

1. Create the docker compose file:

    ``` bash
    cat << EOF > docker-compose.yml    
    services:
      demo1:
        image: docker.io/busybox@sha256:3614ca5eacf0a3a1bcc361c939202a974b4902b9334ff36eb29ffe9011aaad83
        volumes:
          - /mnt/data:/data
        environment:
          - NAME=\${firstname:-World}
          - INTERVAL=\${interval:-60}
        command: >
          sh -c '
            mkdir -p /data/cloudlabs ; env | tee -a /data/cloudlabs/env.out; cat /data/cloudlabs/env.out; head /data/cloudlabs/greetings.out ; tail /data/cloudlabs/greetings.out ; while true ; do sleep \$\${INTERVAL} ; echo hello \$\${NAME} the time is \$\$(date) | tee -a /data/cloudlabs/greetings.out ; done
          '
    EOF
    ```

1. Display the file's content.

    ``` bash
    cat docker-compose.yml
    ```

1. Change to the directory one level higher than your current location (and display it):

    ``` bash
    cd .. && pwd
    ```
    
1. Set an environment variable for your IBM Log Analysis ingestion key. You saved this somewhere safe earlier in the lab.  If you lost track of it, revisit the section *Create an IBM Log Analysis instance" for the steps to retrieve it.

    ``` bash
    read -sp "Log Ingestion Key: " LOG_INGESTION_KEY && echo
    ```

1. Set an environment variable for the hostname of your IBM Log Analysis instance. You saved this somewhere safe earlier in the lab. If you lost track of it, revisit the section *Create an IBM Log Analysis instance" for the steps to retrieve it.

    ``` bash
    read -p "Log Hostname: " LOG_HOSTNAME
    ```

1. Create the contract.

       ``` bash
       cat << EOF > user_data.yaml
       env: |
         type: env
         logging:
           logDNA:
             hostname: ${LOG_HOSTNAME}
             ingestionKey: ${LOG_INGESTION_KEY}
             port: 6514
         volumes:
           data:
             seed: seed-supplied-by-env-persona
         env:
           firstname: King Charles
           interval: "30"
       workload: |
         type: workload
         volumes:
           data:
             filesystem: ext4
             mount: /mnt/data
             seed: seed-supplied-by-workload-persona
         compose:
           archive: $(${LAB_TAR} -czv -C compose . | base64 ${LAB_WRAP}) 
       EOF
       ```

1. Display your contract data. Keep your terminal session handy- later on in the next section you will be directed to copy these contents into the IBM Cloud Web UI. 

    ``` bash
    cat user_data.yaml
    ```

Proceed to the next section to create your Hyper Protect Virtual Servers for IBM Cloud VPC instance.

  
