# Overview and Pre-reqs

## Overview

These labs provide a gentle introduction to the art of creating a [*contract*](https://cloud.ibm.com/docs/vpc?topic=vpc-about-contract_se&interface=ui){target="_blank" rel="noopener"} for a Hyper Protect Virtual Servers instance. The contract you specify to a Hyper Protect Virtual Servers instance specifies the application workload you wish to run and the environmental characteristics such as where to send log messages from your instance.

A best practice is to specify a contract that is encrypted and signed.  These labs start simple-  with an unsigned, plain text contract.  Such a contract would be hard to justify in a production environment, but is a great starting point for learning what is involved with creating a contract.

Then each subsequent lab increases the complexity of the contract such that our final lab guides you to the best practice of creating an encrypted and signed contract.

!!! Warning "Caution- you may incur charges from IBM Cloud for activities performed in these labs "

    You may incur charges for the artifacts created during the lab.  These charges are your responsibility.  Each lab ends with instructions to delete these lab artifacts in order to minimize any costs you might incur.

The flow of the labs can be summarized as:

1. Create prerequisite resources on IBM Cloud such as a Virtual Private Cloud and an IBM Log Analysis instance.

2. Create a Hyper Protect Virtual Server instance using a plain text contract.

3. Create a Hyper Protect Virtual Server instance where the workload provider encrypts their portion of the contract but the environment provider provides their portion in plain text.

4. Create a Hyper Protect Virtual Server instance where both the workload provider encrypts their portion of the contract and the environment provider encrypts their portion of the contract.

5. Create a Hyper Protect Virtual Server instance where both portions of the contract are encrypted and then signed by the environment provider.

6. Deleting the resources created during the labs.


## Prerequisites

These labs assume that you have an IBM Cloud account and have the authorization needed to create the artifacts required by the labs..

All IBM Cloud resources are created with the IBM Cloud web UI from your web browser.

You must have access to a Linux or MacOS system in order to create the contracts required by the lab.

If you have a Windows system you can try to use the [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/){target="_blank" rel="noopener"} or you can create a Linux server elsewhere. We provide instructions on how to create a Linux server using *Hyper Protect Virtual Server for Classic* in a [subsequent section](./helperserver){target="_blank" rel="noopener"} of the lab.


