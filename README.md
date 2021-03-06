# Photoscan-template
Sample templates and scripts that deploys [Agisoft Photoscan](http://www.agisoft.com) product in Azure. This template was designed to work with two storage solutions, one is [Avere vFXT](http://www.averesystems.com) (a Microsoft product) and the other one is [BeeGFS](http://beegfs.io).

For an end to end experience, before you deploy this template you need to deploy one of the storage solutions mentioned before, links for their deployment will be provided at the [Prequisites](#prereq) section. The following diagrams shows a high level view of VM components deployed depending on what storage solution was chosen. 

### Avere vFXT
![image](./docs/media/avere.png)

### BeeGFS
![image](./docs/media/beegfs.png)

This solution is break down into the following components:
* Infrastructure items (Active Directory)
* Jumpboxes so you can connect to the environment
* Photoscan Scheduler (head) node 
* Processing nodes (compute)
* When using Avere vFXT or BeeGFS provided templates, this template will automatically peer to the storage template virtual network

A sample deployment script called Deploy-AzureResourceGroup.sh is provided with this solution and you can use it to help automate your deployment. This is a minimal sample command line example which creates a staging storage account to hold all deployment related files (if you don't provide the storage account name and its resource group, it will create one for you) and deploy using Avere vFXT option (selection is made depending on what parameter file you choose):

```bash
./Deploy-AzureResourceGroup.sh -g myResourceGroup -l eastus -r support-rg -v keyvaultname -p azuredeploy.parameters.avere.json
```

## Prerequisites<a name="prereq"></a>
1. An Azure Key Vault with the following secrets with their values created
    *  adminPassword - this will be your Domain Admin and Local Windows Administrators password
    *  activationCode - this is the Photoscan activation code, make sure you have a valid code or request a trial at their web site.
    
2. To get end to end automation with this template, please deploy one of the storage template solutions this Photoscan template was designed for: 
   * [Avere vFXT](https://github.com/Azure/Avere/tree/master/src/vfxt#experimental-avere-vfxt-controller-and-vfxt---arm-template-deployment)
       *  After you deployed Avere vFXT, make sure you follow the steps documented [here](./docs/AverePostDeploymentSteps.md) to create NFS exports and Namespaces correctly. If you don't have VPN/Express route connectivity to the virtual network where Avere vFXT cluster was deployed, please follow the steps outlined [here](./docs/EstablishingSslVpn.md) in order to establish a SSH tunnel and continue with the post-deployment tasks.
    
   * [BeeGFS](https://github.com/paulomarquesc/beegfs-template).
   
3. There are two parameter files, one for Avere vFXT (`azuredeploy.parameters.avere.json`) and one for BeeGFS (`azuredeploy.parameters.beegfs.json`), please make sure you review the parameter file related to your storage option before starting the deployment.

## Deployment Steps

### Sign in to Cloudshell
1. Open your browser and go to <a href="https://shell.azure.com" target="_new">https://shell.azure.com</a>

1. Sign on with `Microsoft Account` or `Work or School Account` associated with your Azure subscription

    ![image](./docs/media/image1.png)


1. If you have access to more than one Azure Active Directory tenant, Select the Azure directory that is associated with your Azure subscription
    
    ![image](./docs/media/image2.png)

1. If this is the first time you accessed the Cloud Shell, `Select` "Bash (Linux)" when asked which shell to use.

    ![image](./docs/media/image3.png)

    > Note: If this is not the first time and it is the "Powershell" shell that starts, please click in the dropdown box that shows "PowerShell" and select "Bash" instead.

1. If you have at least contributor rights at subscription level, please select which subscription you would like the initialization process to create a storage account and click "Create storage" button.
    ![image](./docs/media/image4.png)

1. You should see a command prompt like this one:
    ![image](./docs/media/image5.png)


### Cloning/Deploying the source sample project
1. Change folder to your clouddrive so any changes gets persisted in the storage account assigned to your cloudshell
   ```bash
   cd ~/clouddrive
   ```
1. Clone this repository with the following git command
   ```bash
   git clone https://github.com/paulomarquesc/photoscan-template.git
   ```
1. Change folder to photoscan-template
   ```bash
   cd photoscan-template
   ```
1. Review and change the parameters files before deploying
   *  azuredeploy.parameters.json

1. Execute the deployment script the template (make sure you change command line arguments appropriately)
    * azuredeploy.json
        ```bash
        ./Deploy-AzureResourceGroup.sh -p azuredeploy.parameters.avere.json -g photoscan-rg -l eastus -s storageaccountname -r storage-account-rg -v mykeyvault
        ```
    > Note: this example will deploy Photoscan using Avere vFXT storage, for BeeGFS, make sure you change the parameter file accordingly.


### Deploying Windows worker nodes instead of Linux OS
By default the template deploys the worker nodes as Linux VMs, for an end-to-end experience, this is the option that will be able to integrate automatically to Avere vFXT or BeeGFS storage if it was deployed using one of the storage templates indicated in the [prerequisites](#prereq) section.
If you want to deploy Windows worker nodes, storage configuration will need to be manual because neither storage solutions are native to Windows at this point. To use Windows compute nodes, please make sure you change the following parameters in any of the parameter file before deploying this template:

`useNfsStorage = "no"`

`useBeeGfsStorage = "no"`

`workerNodesType = "windows"`

> Note: this change can be done on both provided parameter files or in just the one you will use to perform the deployment since the first two parameters listed will make the template ignore any of the storage solutions.

### Template parameters and descriptions

The following parameters are contained on both sample parameter files provided, notice that parameters that are related to Avere vFXT storage option starts with "nfs" and parameters related to BeeGFS starts with "beegfs".

#### azuredeploy.parameters.X.json (where X = avere or beegfs)
* **_artifactsLocation:** Auto-generated container in staging storage account to receive post-build staging folder upload.
* **_artifactsLocationSasToken:** Auto-generated token to access _artifactsLocation.
* **location:** Location where the resources of this template will be deployed to. Default Value: `eastus`
* **adminPassword:** Admin password.
* **useSingleResourceGroup:** Whether or not use multiple resource groups, if using multiple, please change the resource groups manually in the variables section. Default Value: `yes`
* **activationCode:** Photoscan Activation Code.
* **adminUsername:** Name of admin account of the VMs, this name cannot be well know names, like root, admin, administrator, guest, etc.
* **sshKeyData:** SSH rsa public key file as a string.
* **vnetName:** Virtual Network Name. Default Value: `Photoscan-vnet`
* **vnetAdressSpace:** Virtual Network Address Space. Default Value: `10.0.0.0/16`
* **jumpboxSubnetName:** Jumpbox subnet name. Default Value: `Jumpbox-SN`
* **jumpboxSubnetAdressPrefix:** Jumpbox subnet address prefix. Default Value: `10.0.0.0/24`
* **photoscanSubnetName:** Photoscan servers (Scheduler+Node) subnet name. Default Value: `Photoscan-SN`
* **photoscanSubnetAdressPrefix:** Photoscan subnet address prefix. Default Value: `10.0.1.0/24`
* **adSubnetName:** Subnet where Domain Controllers will be deployed to. Default Value: `AD-SN`
* **adSubnetAdressPrefix:** AD subnet address prefix. Default Value: `10.0.2.0/24`
* **dc1Name:** Domain Controller 1 Name. Default Value: `DC-01`
* **dc2Name:** Domain Controller 2 Name. Default Value: `DC-02`
* **dc1IpAddress:** Domain Controller 1 IP Address. Default Value: `10.0.2.4`
* **dc2IpAddress:** Domain Controller 2 IP Address. Default Value: `10.0.2.5`
* **dcVmSize:** Domain Controller VM Size. Default Value: `Standard_DS2_v2`
* **dnsDomainName:** Active Directory FQDN. Default Value: `testdomain.local`
* **adDomainNetBIOSName:** Active Directory NetBIOS domain name. Default Value: `TESTDOMAIN`
* **deployLinuxJumpbox:** Should this template deploy a Linux Jumpbox. Default Value: `yes`
* **windowsJumpboxVmName:** Windows Jumpbox VM Name. Default Value: `wjb-01`
* **linuxJumpboxVmName:** Linux Jumpbox VM Name. Default Value: `ljb-01`
* **windowsJumpboxVmSize:** Windows Jumpbox VM Size. Default Value: `Standard_DS2_v2`
* **linuxJumpboxVmSize:** Linux Jumpbox VM Size. Default Value: `Standard_DS2_v2`
* **windowsJumpboxIpAddress:** Windows Jumpbox VM Ip Address. Default Value: `10.0.0.4`
* **linuxJumpboxIpAddress:** Linux Jumpbox VM Ip Address. Default Value: `10.0.0.5`
* **headServerName:** Photoscan Server (head) name. Default Value: `headnode`
* **headVmSize:** Head node VM Size. Default Value: `Standard_D8S_v3`
* **workerNodesType:** OS type of worker nodes, Linux or Windows. Default Value: `linux`
* **nodeNamePrefix:** Name suffix to be used in the GPU Nodes. Default Value: `workernode`
* **nodeSubnetIpAddressSuffix:** Nodes will have static Ip addresses, this is the network part of a class C subnet. Default Value: `10.0.1`
* **nodeStartIpAddress:** Nodes will have static Ip addresses, this is the start number of the host part of the class C ip address. Default Value: `20`
* **nodeCount:** Number of GPU VM Nodes. Default Value: `5`
* **nodeVmSize:** GPU VM Size. Default Value: `Standard_NC24s_v2`
* **headRoot:** Root path where the projects are located for Server. Default Value: `\\beegfs\beegfsshare\Projects`
* **nodeRoot:** Root path where the projects are located for Nodes. Default Value: `/beegfs/beegfsshare/Projects`
* **dispatch:** Ip address of the photoscan server (head). Default Value: `10.0.1.250`
* **gpuMask:** Decimal represention of how many GPUs will be enabled for processing. E.g. 15 means 1111, that is equal to 4 GPUs. Default Value: `15`
* **windowsPhotoscanDownloadUrl:** Windows binary Photoscan download URL. Default Value: `http://download.agisoft.com/photoscan-pro_1_4_4_x64.msi`
* **linuxPhotoscanDownloadUrl:** Photoscan Linux binaries download URL. Default Value: `http://download.agisoft.com/photoscan-pro_1_4_4_amd64.tar.gz`
* **photoscanInstallPath:** Photoscan installation path. Default Value: `/`
* **photoscanAbsolutePaths:** Use Photoscan absolute paths. 0 = No, 1= Yes. Default Value: `0`
* **useNfsStorage:** Should this template use Nfs storage. If this parameter is set to 'yes', make sure useBeeGfsStorage is set to 'no'. Default Value: `no`
* **useBeeGfsStorage:** Should this template use BeeGfs storage. If this parameter is set to 'yes', make sure useNfsStorage is set to 'no'. Default Value: `yes`
* **storageVnetRG:** Storage Vnet Resoure group name. Default Value: `beegfs-rg-eus`
* **storageVnetName:** Srorage Virtual Network name. Default Value: `beegfs-vnet`
* **sharedNfsStorageHpcUserHomeFolder:** This indicates shared storage mount point on Linux VM nodes for the hpcuser home folder, it will mounted on all Linux nodes. Default Value: `/mnt/beegfshome`
* **homeNfsExportPath:** If useNfsStorage is set to 'yes', this parameter with correct values is mandatory. This is the export path configured in your NFS server for home folder of HPC User. Default Value: `/home`
* **sharedScracthMountPoint:** Folder path where Shared Storage volume will be mounted on Linux VMs.. Default Value: `/beegfs`
* **nfsScratchExportPath:** If useNfsStorage is set to 'yes', this parameter with correct values is mandatory. This is the export path configured in your NFS server to be used by Photoscan for project processing. Default Value: `/data`
* **nfsDnsEntry:** If useNfsStorage is set to 'yes', this parameter with correct values is mandatory. Format is <DNS A Record for NFS servers>,<IP1>,<IP2>,<IP3>,<IPx>. E.g. vfxt,10.0.0.11,10.0.0.12,10.0.0.13. Default Value: ``
* **nfsScratchFolderNfsVersion:** NFS Version used to mount scratch (data) folder. Will be used only when NFS storage is in use. Default Value: `nfs`
* **nfsHomeFolderNfsVersion:** NFS Version used to mount home folder for HPC User. When using BeeGFS, this value must be nfs4, if using Avere vFXT it must be nfs. Default Value: `nfs`
* **nfsScratchMountOptions:** NFS scratch volume mount options, comma separated, no spaces. E.g. noatime,rsize=524288,wsize=524288. Default Value: `defaults`
* **nfsExportPathUNC:** UNC NFS Export path, this value is used by Windows Scheduler (Head) to map the NFS path to a drive letter when using NFS storage. E.g. \\vfxt\!\msazure. Default Value: `\\vfxt\!\msazure`
* **nfsMountType:** Type of NFS mounts, hard or soft. Default Value: `hard`
* **nfsMapDriveLetter:** Drive letter where nfsExportPathUNC will be mapped to. Default Value: `z:`
* **nfsCaseSensitiveLookup:** Sets that NFS will use case sensitive lookups. Default Value: `False`
* **nfsTimeout:** NFS timeout. Default Value: `60`
* **nfsMountRetry:** NFS mount retries. Default Value: `3`
* **nfsDefaultAccessMode:** NFS default access mode. Default Value: `777`
* **nfsWindowsRsizeKb:** Windows NFS Client read size in KB. Default Value: `64`
* **nfsWindowsWsizeKb:** Windows NFS Client write size in KB. Default Value: `64`
* **beeGfsMasterName:** BeeGFS Master node VM name (single label). Default Value: `beegfsmaster`
* **beeGfsMasterIpAddress:** BeeGFS Master Ip address, this will be added as an A record on DNS. Default Value: `192.168.0.4`
* **beeGfsSmbServersVip:** Ip Address of the BeeGFS SMB clients Load Balancer,this will be added as an A record on DNS. Default Value: `192.168.0.55`
* **beeGfsSmbServerARecordName:** BeeGFS A record to be used by Photoscan Server (Head). Default Value: `beegfs`
* **hpcUser:** Hpc user that will be owner of all files in the hpc folder structure. Default Value: `hpcuser`
* **hpcUid:** Hpc User ID. Default Value: `7007`
* **hpcGroup:** Hpc Group. Default Value: `hpcgroup`
* **hpcGid:** Hpc Group ID. Default Value: `7007`

## References
Avere - http://www.averesystems.com

BeeGFS - https://www.beegfs.io

Avere Deployment Template - https://github.com/Azure/Avere/tree/master/src/vfxt#experimental-avere-vfxt-controller-and-vfxt---arm-template-deployment

BeeGFS Deployment Template - https://github.com/paulomarquesc/beegfs-template

Agisoft Photoscan - http://www.agisoft.com/