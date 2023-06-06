# Prepare the contract 

## Ensure necessary environment variables are set

1. Go to a command prompt on your prep system

2. You should have each of these environment variables set on your prep system:

    ``` bash
    echo LAB_WORKDIR is ${LAB_WORKDIR}
    echo LAB_TAR is ${LAB_TAR}
    echo LOG_INGESTION_KEY is ${LOG_INGESTION_KEY}
    echo LOG_HOSTNAME is ${LOG_HOSTNAME}
    ```

    If any of the above commands do not display a value after the _is_ then revisit [the section on setting environment variables](../prereqs/setup.md){target="_blank" rel="noopener"}.

## Make a new directory for Lab 1

1. Make a fresh directory structure and change in to it:

    ``` bash
    mkdir -p ${LAB_WORKDIR}/lab1 && cd ${LAB_WORKDIR}/lab1
    ```

## Create a Docker compose file to specify the application workload

1. Create another directory to hold a Docker compose file and switch to it:

    ``` bash
    mkdir compose && cd compose
    ```

2. Create the docker compose file:

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

3. Display the file's content.

    ``` bash
    cat docker-compose.yml
    ```

## Create the contract

1. Change to the directory one level higher than your current location (and display it):

    ``` bash
    cd .. && pwd
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

  
