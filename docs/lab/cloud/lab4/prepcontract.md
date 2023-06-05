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

    If any of the above commands do not display a value after the _is_ then revisit [sections](../prereqs/setup.md){target="_blank" rel="noopener"} on setting environment variables.

## Make a new directory for Lab 4

1. Make a fresh directory structure and change in to it:

    ``` bash
    mkdir -p ${LAB_WORKDIR}/lab4 && cd ${LAB_WORKDIR}/lab4
    ```

## Create a Docker compose file to specify the application workload

1. Create the following directory structure and then switch to the directory that will hold the docker compose file:

    ``` bash
    mkdir -p {environment,workload/compose} && cd workload/compose
    ```

2. Create the docker compose file (Note: this is the same docker compose file from the first three labs):

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

3. **Optional**.  This command will show that you are indeed using the same docker compose file in this lab (lab 4)  as you did in the previous lab (lab 3).  We could have had you just copy that file over instead of using the *cat* command in the prior step.


    ``` bash
    diff --report-identical-files \
      ${HOME}/cloudlabs/lab3/workload/compose/docker-compose.yml \
      ${HOME}/cloudlabs/lab4/workload/compose/docker-compose.yml
    ```


4. Display the file's content.

    ``` bash
    cat docker-compose.yml
    ```

## Create a convenience script to encrypt the workload section

1. Backup one directory level:

    ``` bash
    cd ..
    ```
    
2. Create this convenience script to encrypt the workload portion of the contract:
 
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

## Create a convenience script to encrypt the environment  section

1. Change to the directory used for the environment section display it:

    ``` bash
    cd ../environment && pwd
    ```
    
2. Create this convenience script to encrypt the environment portion of the contract:

    ``` bash
    cat << EOF > flow.env
    # Create the env section of the contract and add the contents in the env.yaml file.
    ENV_PLAIN="./env.yaml.plaintext"
    ENV="env.yaml"
    
    echo "  type: env
      logging:
        logDNA:
          hostname: \${LOG_HOSTNAME}
          ingestionKey: \${LOG_INGESTION_KEY}
          port: 6514
      env:
        firstname: Princess Catherine
        interval: \"30\"
      volumes:
        data:
          seed: seed-supplied-by-env-persona" > \${ENV_PLAIN}
    
    cat ./pubSigningKey.yaml >> \${ENV_PLAIN}

    # Download certificate to encrypt contract for Hyper Protect Container Runtime:
    HPCR_rev=10
    CONTRACT_KEY=./ibm-hyper-protect-container-runtime-1-0-s390x-\${HPCR_rev}-encrypt.crt
    curl https://cloud.ibm.com/media/docs/downloads/hyper-protect-container-runtime/\$CONTRACT_KEY > \$CONTRACT_KEY
    
    
    # Use the following command to create a random password:
    PASSWORD_ENV="\$(openssl rand 32 | base64 \${LAB_WRAP})"
    
    #  Use the following command to encrypt password with the Hyper Protect Container Runtime Contract Encryption Key:
    ENCRYPTED_ENV_PASSWORD="\$(echo -n "\$PASSWORD_ENV" | base64 -d | openssl rsautl -encrypt -inkey \$CONTRACT_KEY -certin | base64 \${LAB_WRAP} )"
    
    # Use the following command to encrypt env.yaml with a random password:
    ENCRYPTED_ENV="\$(echo -n "\$PASSWORD_ENV" | base64 -d | openssl enc -aes-256-cbc -pbkdf2 -pass stdin -in "\$ENV_PLAIN" | base64 \${LAB_WRAP})"
    
    # Use the following command to get the encrypted section of the contract:
    ENV_ENCRYPTED="hyper-protect-basic.\${ENCRYPTED_ENV_PASSWORD}.\${ENCRYPTED_ENV}"
    
    echo ""
    echo "See `pwd`/env.yaml.plaintext to see what was encrypted for the env section of your contract"
    echo ""
    
    echo "\$ENV_ENCRYPTED" > ../\$ENV
    EOF
    ```

## Create convenience scripts to facilitate signing the contract

1. Create the following convenience script that will create an RSA key pair that you will use to sign the contract. 

    ``` bash
    cat << EOF > flow.prepare
    # Use the following command to generate key pair to sign the contract 
    openssl genrsa -aes128 -passout pass:test1234 -out private.pem 4096
    openssl rsa -in private.pem -passin pass:test1234 -pubout -out public.pem

    # The following command is an example of how you can get the signing key:
    key=\$(awk -vRS="\n" -vORS="\\\\\n" '1' public.pem)
    # echo "  signingKey: \"\${key%\\\\n}\"" > environment/pubSigningKey.yaml
    printf "%s" "  signingKey: \"\${key%\\\\n}\"" > environment/pubSigningKey.yaml
    EOF
    ```

2. Create the following convenience script that will sign the encrypted contract.

    ``` bash
    cat << EOF > flow.signature
    # combine workload and environment
    cat workload.yaml env.yaml | tr -d '\n' > contract.yaml

    # Sign the combination from workload and env being approved
    echo \$( cat contract.yaml | openssl dgst -sha256 -sign private.pem -passin pass:test1234 | openssl enc -base64) | tr -d ' ' > signature.yaml

    # Create user data and add signature:
    echo "workload: \$(cat workload.yaml)
    env: \$(cat env.yaml)
    envWorkloadSignature: \$(cat signature.yaml)" > user_data.yaml
    
    echo ""
    echo "import `pwd`/user_data.yaml into User Data or copy and paste from below:"
    echo ""
    
    cat user_data.yaml
    EOF
    ```

## Create and run helper script to produce the encrypted and signed contract

1. Create this helper script that you will use to create the signed and encrypted contract:

    ``` bash
    cat << EOF > makeContract
    . ./flow.prepare
    cd workload
    . ./flow.workload
    cd ../environment
    . ./flow.env
    cd ..
    . ./flow.signature
    EOF
    ```

2. Create the contract:

    ``` bash
    . ./makeContract
    ```
   
3. Display your contract data:

    ``` bash
    cat user_data.yaml
    ```

Proceed to the next section to create your Hyper Protect Virtual Servers for IBM Cloud VPC instance.

  
