# **IPsec VPN Connection between OCI & Azure**
 

**Statement of Requirements**

Currently, for the following connections there are documentations,

- OCI and Azure is using Fastconnect and Express route
- Using Libreswan.

There is no documentation for establishing private connection (IPsec) between OCI and Azure.

This document serves as a design implementations and components of building private connection between OCI and Azure



**Diagram**

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.001.png)

**Requirements**

- Non-Overlapping CIDR IP address space Azure and OCI.

**Azure**

- VNET (Virtual Network) – 11.0.0.0/16
- Subnet-11.0.0.0/24
- Gateway Subnet – 11.0.1.0/27
- VPN Gateway
- Local Gateway (OCI Tunnel IP)
- Routing Table
- Network Security Group
- Virtual Machine

` `**OCI**

- VCN – 10.0.0.0/16.
- Subnet-10.0.0.0/24.
- DRG
- CPE – (Azure side VPN Gateway IP).
- Internet Gateway
- Routing table.
- Security lists / NSG.
- Virtual Machine



**High Level steps**

**Azure side**

- Create Vnet and Subnet.
- Create and configure Vnet gateway.
- Create Local gateway.
- Create IP sec connection.
- Create Route table and Routes, Also allow traffic on NSG.
- Test the connectivity.

**OCI Side**

- Create VCN and subnet.
- Create and configure DRG.
- Create CPE.
- Create IPsec tunnel(IKEV2).
- Configure routes and Security list
- Test the connectivity

**Detailed Procedure- Step by Step**

**Azure Network Configuration**

` `**Creating Vnet and Subnet**

- Select Create a resource in the upper left-hand corner of the portal.
- In the search box, enter Virtual Network. Select Virtual Network in the search results.
- In the Virtual Network page, select Create.
- In Create virtual network, enter or select this information in the Basics tab:
- Refer <https://docs.microsoft.com/en-us/azure/virtual-network/quick-create-portal>, for more info about how to create vnet in Azure.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.002.png)

**Create VNET Gateway**

In this step, you create the virtual network gateway for your VNet. Creating a gateway can often take 45 minutes or more, depending on the selected gateway SKU.

Create a virtual network gateway using the following values:

- **Name:**AzrVPNGW
- **Region:**UK West
- **Gateway type:**VPN
- **VPN type:**Route-based
- **SKU:**VpnGw1
- **Generation:**Generation1
- **Virtual network:**Azr-VPN
- **Gateway subnet address range:**11**.**0.1.0/27
- **Public IP address:**Create new
- **Public IP address name:**VNet1GWpip
- **Enable active-active mode:**Disabled
- **Configure BGP:**Disabled
- From the [Azure portal](https://portal.azure.com/), in **Search resources, services,** Locate **Virtual network gateway** in the search results and select it.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.003.png)

- On the **Virtual network gateway**page, select **+ Add**. This opens the **Create virtual network gateway**

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.004.png)

- On the **Basics** tab, fill in the values for your virtual network gateway.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.005.png)

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.006.png)

Click on Review + create

Verify the details and click on Create. Sit back and relax it will take almost 45 minutes.

**OCI Network Configuration**

**Create VCN and Subnet**

- Open the navigation menu, click Networking, and then click Virtual Cloud Networks.
- Ensure that the Sandbox compartment (or the compartment designated for you) is selected in the Compartment list on the left.
- Click Create VCN using below details
- **Name:** OCI-VCN
- **Region:** Amsterdam
- **CIDR:** 10.0.0.0/16

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.007.png)

**Create DRG**

- Open the navigation menu. Go to Networking and click Customer Connectivity.
- Choose a compartment you have permission to work in (on the left side of the page). The page updates to display only the resources in that compartment. If you're not sure which compartment to use, contact an administrator.
- Click Create Dynamic Routing Gateway.
- Enter the following items: 
  - Name: OCI-DRG
  - Compartment: Select required compartment.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.008.png)

**Attach DRG to VCN**

- Open navigation menu, go to Networking and click Virtual cloud networks
- Select the VCN, OCI-VCN.
- Select Dynamic Routing gateways attachment
- Click on Create DRG attachment > Select the DRG we have created – OCI-DRG

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.009.png)

**Create CPE**

Once VPN Gateway has been created in Azure, Note the Public IP address of it and we will use it to create the CPE.

- Open the navigation menu. Go to Networking and click Customer Connectivity.
- Click on Customer Premises Equipment and select Create.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.010.png)

- Create the CPE as below. Public IP address of CPE is IP address of Azure VPN gateway.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.011.png)


**Create IP sec connection-OCI**

Since we do not know the Public IP address of the DRG, We will have to create a IP sec tunnel and use the tunnel public IP address as local gateway in Azure.

OCI side we have created DRG and CPE so we are ready to establish a connection.

- Open the navigation menu. Go to Networking and click Customer Connectivity.
- Click Site-to-Site-VPN (IPsec).

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.012.png)

Create the IPsec connection using below details

- CPE: Select the CPE
- DRG: Select the DRG
- Static route: Input the IP range of Azure VNet, In our case it is 11.0.0.0/16.
- Tunnel IKE Version: IKEV2

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.013.png)

- Click on Create IP sec connection.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.014.png)

Now you got Public IP addresses of two tunnels. Let’s go to Azure and configure them as Local network gateway.

**Create Local Network Gateways – Azure**

- On Azure portal, Seach for Local network gateways and select it.
- Click on create.
- IP Address: OCI tunnel1 public IP.
- Address Space: OCI VCN range.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.015.png)

- Click on Create.

Now we are all set to create IP sec connection  from Azure side.

**Crate IP SEC connection - Azure**

- From home menu, Search for connections service and click on it.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.016.png)

- Click on create
- Preshared key: Copy the key from OCI tunnel information.
- DPD timeout: Set as per your need, ex: 600 secs.
- Connection mode: Default.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.017.png)

- Click on Review+create and click on create.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.018.png)

**Connection is UP!**

Now to test the communication, Have to allow the traffic and route it towards IPsec.

**Create route rules:**

**Azure Side:**

Create a route table, add routes and associate it with the subnet.

- In Home menu, search for route table and click on it
- Click on create, and create like below.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.019.png)

- Add a route rule

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.020.png)


- Address prefix: OCI VCN CIDR range

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.021.png)

- Associate with the subnet.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.022.png)

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.023.png)

Create NSG and add rules to allow OCI network

- We can do this while creating the VM. The rule set for NSG is below.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.024.png)

**Create Route rules to default RT of the VCN – OCI Side**

- Open the VCN menu, Click on route tables and select default route table.
- Add a route rule as below.

![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.025.png)

**Testing the communication.**

- OCI VM: 10.0.0.239
- Azure VM: 11.0.0.4

` `![](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.026.png)

**Hola !! Communication test is passed ![(smile)](images/Aspose.Words.aa7c2493-b54f-4201-a7c6-e46e6c3ee85d.027.png)**


Thanks to Adamshaffi Shaik for the lab setup !

Thank you :) 
