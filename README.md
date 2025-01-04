![image](https://github.com/user-attachments/assets/0b658f02-1955-4bdb-98f4-7761163c090f)
# Deploying-Web-Server-within-Azure-VM
This tutorial outlines the implementation of an Ubuntu web server within an Azure virtual machine.
## Environments and Technologies Used
+ Microsoft Azure (Virtual Machines)
+ Remote Desktop
+ Bash
## Operating Systems Used
+ Ubuntu Server 22.04 LTS
## Deployment and Configuration Steps
In this lab, we're going to create a VM within a VNET in order to deploy our web server. This will require a Microsoft Azure account. First, you'll need to create a resource group. From your account homepage, select resource groups then create. A standard process of naming our resources so that we maintain organization is by using the format: type-region-application. An example could be RG-USE-Nextcloud (USE=United States East). Click review+create then create. Your resource group is now created.

<img width="959" alt="SS1" src="https://github.com/user-attachments/assets/e6d4fc4b-1213-4337-b7ea-6032a0ef3c63" />

Next were're going to create a virtual network and subnet. A virtual network is a logical network that enables different resources to securely communicate with each other as if they were physically connected. Open your resource group and click create to search for and add a virtual network.

<img width="957" alt="SS2" src="https://github.com/user-attachments/assets/c54e4573-ec60-4f2a-b449-3a50d9afbfd5" />

Following the same format I've named it VNET-USE-Nextcloud. You'll need to give an address space for the IPs of the virtual network. Change default ipv4 address space 10 172.10.0.0/16. This is an range of public ATT IP addresses. Next add a subnet, which is a smaller network inside a large network that makes for more efficient network routing. I've named mine SNET-USE-Nextcloud. Now give it an address range which should be a subset of the address space of the VNET. Review your parameters and click create.

<img width="956" alt="SS3" src="https://github.com/user-attachments/assets/af703a9c-cc48-403c-824a-daaa2c2e63ad" />

To protect the subnet, create a Network Security Group. An NSG helps filter traffic to and from Azure resources within the VNET. This will allow you to allow or deny inbound and outbound traffic. Once again, open your resource group and create a new resource by searching network security group. I've named mine NSG-USE-Nextcloud. We will now assign the default inbound and outbound created by Azure in the NSG to the subnet. Open your subnet and select the NSG you just created under the network security group field. Next, we will deploy Bastion to connect to our VM. Bastion allows for web-based RDP access to the VNET and VM. We'll create this deployment in order to connect to our virtual machine using SSH. First, create a subnet for Bastion within the VNET and name it AzureBastionSubnet. Use the suggested starting address and make it size /24. Now, create a Bastion resource within the resource group. I've named mine BASTION-USE-Nextcloud. Select the Nextcloud VNET and the Bastion subnet when configuring this Bastion resource. I've changed the public IP address name of the Bastion resource to BASTIONIP-USE-Nextcloud. Deployment may take a while so you can be continue on with next steps while that loads.

<img width="959" alt="SS4" src="https://github.com/user-attachments/assets/fdf6f125-78d2-4621-8985-8e1376bfad1d" />

Add an Ubuntu server VM to the resource group. The one I am using is Ubuntu Server 22.04 LTS. Name it using the same format. It is essential that the VM is in the same region as the VNET. Also keep in mind that we are using default avaliability zone 1. The Public IP has to be deployed in the same availability zone as the VM. We are installing a basic Nextcloud server for size, you can choose B1s (click see all sizes to select). Choose SSH public key for authentication and create a username. You can change the key pair name to something more understandable such as "VM-USE-Nextcloud_SSHkey". We don't want the SSH port to be available over the internet since we're connecting via Bastion so change the public inbound ports to none. Under networking change the public IP field to "none". Review your parameters and create the Ubuntu VM. Download the private key and allow the VM to finish deploying.

<img width="959" alt="SS5" src="https://github.com/user-attachments/assets/9e044759-264e-411d-9eab-65403020a0f7" />

Next, we will connect to the VM via SSH to install Nextcloud. In your resource group, open up the VM and take note of its private IP address. Click connect then connect via Bastion. Type in your previously created username then choose "SSH private key from local file" for authentication type. Open directory then choose the SSH key you previously downloaded. You may need to allow pop ups to successfully connect. Congrats! you're now connected to your VM using SSH via Bastion. Install Nextcloud through the Bash command line by typing `sudo snap install nextcloud`.  Once it's downloaded create an admin account with whatever password you choose by typing `sudo nextcloud.manual-install admin password`. Now we will create a simple self-signed certificate by typing `sudo nextcloud.enable-https self-signed`. Once it finishes, you can exit from SSH by typing `exit` into the Bash command line then close Bastion.

<img width="959" alt="SS6" src="https://github.com/user-attachments/assets/8dcb7593-bfbb-4d36-bd10-a79cfe5f3512" />
<img width="958" alt="SS7" src="https://github.com/user-attachments/assets/0a78967b-c85e-46b1-b2a1-5a850231d2f4" />

In order to access the Nextcloud instance from the web, we will publish an IP. In the Azure interface, open your VM then navigate to networking settings then to IP configuration settings. Click on ipconfig1 and associate a new public IP address by clicking "create new". Mine is named VMIP-USE-Nextcloud. Choose standard SKU then save. You can go back to your VM to verify that it now has a public IP address. To access the IP through the web, create a rule in the NSG that allows inbound HTTPS traffic. To do this, open a new tab in your browser and navigate to www.whatsmyip.com. Copy that IP then create the inbound port rule from the networking settings in the VM. 

<img width="958" alt="SS8" src="https://github.com/user-attachments/assets/8daa3cbf-3ae9-4806-a8ca-69cbb7e39452" />

Change source to IP address then enter the IP address you just copied. For destination IP, enter the VM private IP address we took note of earlier. Change service to "HTTPS". Name the rule then add. Navigate to VM network settings and copy the public IP address. In a new browser tab type "https://" and that IP. The Browser will alert you that this might not be safe because we used a self-sign certificate. Proceed anyway and it will take you to Nextcloud.

<img width="959" alt="SS9" src="https://github.com/user-attachments/assets/23c6bf8b-b5e3-4364-94a9-add0edbc5e63" />

Now we will create a DNS entry for the public IP. DNS allows us to access websites and services without the need for memorizing IP addresses. From your resource group, open your VM public IP address then navigate to configuration. Give it a DNS name label, you can choose whatever you want. You should be able to see this DNS name in your VM in the Azure interface. Connect to your VM and enter login credentials. To connect the DNS name to Nextcloud, write `sudo nextcloud.occ config:system:set trusted_domains 1 -- value=(the DNS name you chose)`. 
<img width="950" alt="SS11" src="https://github.com/user-attachments/assets/00781592-1d6b-420b-b3fa-729efb32c1d1" />

Exit the SSH connection. Open a new tab and enter the https:// and the DNS/website name. Accept the self-sign certificate then proceed. Congrats! Nextcloud is up and running. Login with the admin credentials you created earlier. Remember to logout when done to follow general cybersecurity guidelines.
<img width="715" alt="SS12" src="https://github.com/user-attachments/assets/a20016ff-ff6d-4a81-adfd-06521b1f0567" />
