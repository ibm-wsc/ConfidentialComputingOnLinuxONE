# Prepare the contract 

1. Go to a command prompt on your prep system

1. Make a fresh directory and change in to it:

    ``` bash
    mkdir -p ${HOME}/cloudlabs/lab2 && cd ${HOME}/cloudlabs/lab2
    ```

1. Create another directory to hold a docker compose file and switch to it:

    ``` bash
    mkdir -p workload/compose && cd workload/compose
    ```

1. Create the docker compose file (Note: this is the same docker compose file from the first lab):

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

1. **Optional**.  This command will show that you are indeed using the same docker compose file in this lab (lab2)  as you did in the previous lab (lab1).  We could have had you just copy that file over instead of using the *cat* command in the prior step.


    ``` bash
    diff --report-identical-files \
      ${HOME}/cloudlabs/lab1/compose/docker-compose.yml \
      ${HOME}/cloudlabs/lab2/workload/compose/docker-compose.yml
    ```

1. Display the file's content.

    ``` bash
    cat docker-compose.yml
    ```

1. Change to the directory one level higher than your current location (and display it):

    ``` bash
    cd .. && pwd
    ```
    
1. Check to see if the environment variable for your IBM Log Analysis instance is still set from the prior lab:

    ``` bash
    echo ${LOG_INGESTION_KEY}
    ```

1. Skip this instruction if it is already set; otherwise, set an environment variable for your IBM Log Analysis ingestion key

    ``` bash
    read -sp "Log Ingestion Key: " LOG_INGESTION_KEY && echo
    ```

1. Check to see if the environment variable for the hostname of your IBM Log Analysis instance is still set from the prior lab:

    ``` bash
    echo ${LOG_HOSTNAME}
    ```

1. Skip this instruction if it is already set; otherwise, set an environment variable for the hostname of your IBM Log Analysis instance

    ``` bash
    read -p "Log Hostname: " LOG_HOSTNAME
    ```

1. Create this convenience script to encrypt the workload portion of the contract:
 
    ``` bash
    cat << EOF > flow.workload
    # Create the workload section of the contract and add the contents in the workload.yaml file.
    
    WORKLOAD_PLAIN=./workload.yaml.plaintext
    WORKLOAD=workload.yaml
    
    echo "  type: workload
      volumes:
        data:
          filesystem: ext4
          mount: /mnt/data
          seed: seed-supplied-by-workload-persona
      compose:
        archive: \$(\${LAB_TAR} -czv -C compose . | base64 \${LAB_WRAP})" > \${WORKLOAD_PLAIN}
    
    # Download certificate to encrypt contract for Hyper Protect Container Runtime:
    HPCR_rev=10
    CONTRACT_KEY=./ibm-hyper-protect-container-runtime-1-0-s390x-\${HPCR_rev}-encrypt.crt
    curl https://cloud.ibm.com/media/docs/downloads/hyper-protect-container-runtime/ibm-hyper-protect-container-runtime-1-0-s390x-\${HPCR_rev}-encrypt.crt > \${CONTRACT_KEY}
    
    
    # Use the following command to create a random password:
    PASSWORD_WORKLOAD="\$(openssl rand 32 | base64 \${LAB_WRAP})"
    
    # Use the following command to encrypt password with the Hyper Protect Container Runtime Contract Encryption Key:
    ENCRYPTED_WORKLOAD_PASSWORD="\$(echo -n "\$PASSWORD_WORKLOAD" | base64 -d | openssl rsautl -encrypt -inkey \$CONTRACT_KEY -certin | base64 \${LAB_WRAP})"
    
    # Use the following command to encrypt the workload.yaml file with a random password:
    ENCRYPTED_WORKLOAD="\$(echo -n "\$PASSWORD_WORKLOAD" | base64 -d | openssl enc -aes-256-cbc -pbkdf2 -pass stdin -in "\$WORKLOAD_PLAIN" | base64 \${LAB_WRAP})"
    
    # Use the following command to get the encrypted section of the contract:
    WORKLOAD_ENCRYPTED="hyper-protect-basic.\${ENCRYPTED_WORKLOAD_PASSWORD}.\${ENCRYPTED_WORKLOAD}"
    
    echo ""
    echo "See `pwd`/workload.yaml.plaintext to see what was encrypted for the workload section of your contract"
    echo ""
    
    echo "\$WORKLOAD_ENCRYPTED" > ../\$WORKLOAD
    EOF
    ```

1. Run the script you just created:

    ``` bash
    . ./flow.workload
    ```

1. Back up one directory:

    ``` bash
    cd ..
    ```

1. Display the encrypted workload section you just created:  

    ``` bash
    cat workload.yaml
    ```

1. Add a plaintext environment section to the encrypted workload section:

    ``` bash
    cat << EOF > user_data.yaml
    workload: $(cat workload.yaml)
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
        firstname: Queen Camilla
        interval: "30"
    EOF
    ```
   
1. Display your contract data:

    ``` bash
    cat user_data.yaml
    ```

Proceed to the next section to create your Hyper Protect Virtual Servers for IBM Cloud VPC instance.

  
