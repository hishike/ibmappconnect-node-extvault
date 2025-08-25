# IBM App Connect Enterprise 12.0.9.0 (or above) external vault setup for Linux

This step-by-step will help you creating an external vault that can store credentials safely using an Integration Node. Note that the Integratio Node must have Integration Servers linked to it. **These steps don't work with Standalone Integration Servers.**

## Common Issue for Virtual Machines or non-graphic interface ACE operations

Before we start the configuration, it is interesting to note that there are some particularities regarding the software.

If you are trying to execute ACE's command lines on the terminal and get *library issues*, it can be related to ACE not finding the mqsiprofile source and you might need to find the mqsiprofile and set it up properly. You can find que mqsiprofile by using

```
find / -name mqsiprofile 2>/dev/null
```

And then setting up the source with the correct path, like the example below:

```
source /home/itzuser/ibm-ace-12.0.9/ace-12.0.9.0/server/bin/mqsiprofile
```

## Creating an Integration Node, Integration Server, External Vault and Credentials:

If you don't have an existing Integration Node, you can create it by using the command below:

```
mqsicreatebroker integration_Node_Name
```

1 - To create an Integration Server, execute the command below while replacing the integration_Node_Name with the one you previously created or will use and define the new Integration Server name:

```
mqsicreateexecutiongroup integration_Node_Name -e integration_Server_Name
```

2 - Create a new directory that will be use to store the new vault and specific credentials:

```
mkdir -p /home/itzuser/ext-vault
```

3 - Create the vault referencing the vaultrc-location, which is the directory created on the previous step and setting a password to access it:

```
mqsivault --vaultrc-store-ext-key --ext-vault-dir /home/itzuser/ext-vault --ext-vault-key password --vaultrc-location /home/itzuser/ext-vault
```

4 - Use the command export so ACE searches the external vault and credentials in the specific directory instead of the default place:

```
export MQSI_VAULTRC_LOCATION=/home/itzuser/ext-vault
```

5 - Create a vault key password while referencing the vault folder. On the example below, we use "password", but you should set a proper password to your external vault:

```
mqsivault --ext-vault-dir /home/itzuser/ext-vault/ --ext-vault-key password --create
```

6 - Set up the credential you want to store by setting up the username and password. Note that for this step, you should create the credential name as basic_auth_cred or a similar name due to it being used on further steps:

```
mqsicredentials --ext-vault-dir /home/itzuser/ext-vault --vaultrc-location /home/itzuser/ext-vault --create --credential-type http --credential-name basic_auth_cred --username admin --password password
```

7 - Set up an admin and password for your web interface:

```
mqsiwebuseradmin -c -u admin -r admin -a password **integrationNode**
```

8 - Find and access the Integration Node node.conf.yaml file:

```
find / -name node.conf.yaml 2>/dev/null
```

```
nano /home/itzuser/aceconfig/components/integration_Node_Name/node.conf.yaml
```

8.1 - Search for "externalDirectoryVault", uncomment and add your ext-vault-dir path
Then search for #basicAuth, remove the # and change the status to true. Save and exit

9 - Access the Integration Server server.conf.yaml with the following command line:
```
find / -name server.conf.yaml 2>/dev/null
```

```
nano /home/itzuser/aceconfig/components/integration_Node_Name/servers/integration_Server_Name/server.conf.yaml
```

Add at the bottom of the file:

```
ServerCredentials:
    local:
      StaticBasicAuthentication:
        username: 'admin'
        password: 'password' 
```

After this step you might set up your APIs and Policies according to your business needs.
