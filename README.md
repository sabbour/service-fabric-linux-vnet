# Secure Service Fabric on Linux inside an existing Virtual Network

This guide will help you create a Secured Linux Service Fabric cluster that runs inside an **existing** Virtual Network and Subnet using the Azure CLI.

> The template is based on the [sample published here](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master) and is modified to provision into an existing Virtual Network and Subnet.

>This guide assumes you already have a Virtual Network created with a Subnet where you want to deploy Service Fabric. If not, you need to create the Virtual Network and Subnet first and specify the values below.

## Steps

1. Specify variable values

```sh
declare rg=sf-rg # Resource Group Name
declare location=westeurope # Region
declare sfName=sfcluster # Service Fabric cluster name
```

2. Create Resource Group

```sh
az group create -n $rg -l $location
```

3. Modify the **parameters.json** file and replace:

    * ``clusterName`` with your Service Fabric cluster name
    * ``clusterLocation`` with your region name
    * ``adminUserName`` with your VM admin username
    * ``adminPassword`` with your VM admin password
    * ``existingVirtualNetworkNameRGName`` with your existing Virtual Network Resource Group name
    * ``existingVirtualNetworkName`` with your existing Virtual Network name
    * ``existingSubnetName`` with your existing Virtual Network name
    * ``existingSubnetPrefix`` with your existing Virtual Network name

4. Create the folder to store the certificates

```sh
mkdir -p certs
```

5. Create the cluster and generate a certificate

```sh
az sf cluster create -g $rg -l $location \
--certificate-output-folder certs \
--certificate-subject-name "$sfName.$location.cloudapp.azure.com" \
--template-file template.json --parameter-file parameters.json
```

6. Connect to the cluster

    * On MacOS/Linux:
        ```sh
        sfctl cluster select --endpoint https://"$sfName.$location.cloudapp.azure.com":19080 --pem /path/to/certificate.pem --no-verify
        ```
    
    * On Windows:
        ```sh
        sfctl cluster select --endpoint https://"$sfName.$location.cloudapp.azure.com":19080 --cert /path/to/certificate.pfx --no-verify
        ```