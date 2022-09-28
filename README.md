# SIEM_AZURE
In this tutorial we will setup Azure Sentinel (SIEM) and connect it to a live virtual machine acting as a honey pot. We will observe live attacks (RDP Brute Force) from all around the world. We will use a custom PowerShell script to look up the attackers Geolocation information and plot it on the Azure Sentinel Map.
STEPS:
1. Create Azure Subscription
First start by setting up a free Azure account (https://azure.microsoft.com/en-us/free/). A $200 Azure credit will be assigned to your account for up to 30 days.
2. Create Virtual Machine
> This is the machine that will be exposed on the internet. To create one, open -> https://portal.azure.com/#home and input “Virtual machines” and select it on the search bar.
> In the “Create a virtual machine” screen, input the following info on the “Basics” tab:
> Instance details: Virtual machine name: honeypot1-vm. Leave everything as default
> There is regional weirdness with Sentinel. For whatever reason, I could not add Sentinel to a US West 3 workspace even though the documentation said it Sentinel was "non-regional"...anyway, I used UK South and it worked like a charm.
> Network Interface: NIC network security group: Click on the “Advanced” toggle. Configure network security group: Click on the “Create new” button. In the “Create network security group” screen remove the default “Inbound rules”. Click on “Add an inbound rule” and input the following. Click on the “Add” button. This will allow all traffic from the internet to enter our VM.
> Destination port ranges: *
Priority: 100
Name: DANGER_ANY_IN
> Click on “Review and create”.
3. Allow all in Firewall
5. Create Log Analytics Workspace
> The purpose of creating a Log Analytics workspace is to ingest logs from the VM. The window events logs from the VM will be ingested to the workspace and it will be converted to custom logs which will contain geographic information. This geographic information allows us to determine the location of where the attacks are coming from. Also, our SIEM (Azure sentinel) will connect to this workspace to be able to display the attacks through a map.
> On the “Create Log Analytics Workspace” screen, click on the “Create” button. 
Input the following: 
Resource group: honeypot1-vm_group (Resource group name created from -> 2) Create a Virtual Machine). 
Name: law-honeypot1. 
Click on “Review + Create” button.
6. Enable gathering VM logs in Microsoft Defender for Cloud
> Find and click on "Environment Settings" in lefthand toolbar 
> Find and click on the dropdown arrow immediately next to your Azure subscription to reveal the NAME of your workspace (this is a critical detail that cost me a lot of time and pain, also bear in mind everything has to be deployed in order for this step to work)
> Click on the workspace name to open its settings
> In settings, disable "SQL servers on machines" 
> In settings, enable "Servers"
> click the save button in the top left next to the search bar
> click on "Data Collection" in the lefthand toolbar
> Select "All Events" and save by clicking on the "Save" button
> jump back to Josh's awesome video and connect the VM to your LAW
7. Connect Log Analytics to VM
8. Setup Azure Sentinel
9. Log into VM with Remote Desktop (fail 1 logon)
> Search for and select “Virtual machines” in the Search bar. Select the VM, “honeypot1-vm” .
> Copy the “Public IP address”.
> Click your Start Menu and type and open “Remote Desktop Connection”. Input the IP address in the “Computer” field and click on “Connect”.
> Type in an incorrect username and password and click “OK”.
> Accept the certification warning. The VM will now begin to open.
10. Observe Event Viewer Logs in VM
11. Turn of Windows Firewall on VM
12. Download PowerShell Script
13. Get Geolocation.io API Key
14. Run Script To get Geo Data from attackers
15. Create custom log in LAW to bring in our custom log
16. Create custom fields/extract fields from raw custom log data
17. Testing Extracts
18. Setup map in sentinel with Latitude and Longitude (or country)
> We will now setup a map within Sentinel which is currently linked to FAILED_RDP_WITH_GEO_CL
> In Azure, search for and select “Microsoft Sentinel” in the Search bar. Select the workspace, “law-honeypot1”
> Click on “Workbooks” and click on “Add workbook”
> Click on “Edit” and remove the two default widgets
> Click on “Add” -> “Add query”. Paste the following query:
> FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF
| where destinationhost_CF != "samplehost"
| where sourcehost_CF != ""
> Set “Visualization” to “Map”, and “Map Size” to “Full”
> In the “Map Settings” set the fields to the following:
“Latitude”: latitude_CF
“Longitude”: longitude_CF
“Size by”: event_count
“Metric Label”: label_CF
> Click on “Done Editing”. Set “Auto refresh” to “5 minutes”
19. Fixing Map plot sizes
20. Final check on map
21. Final thoughts
> By going through this tutorial its evident that as soon as anything is put up on the internet (for this example -> an open VM), people from around the world will attempt to access it. You become a target of opportunity if anything is open. Secondly, by going through the logs of the VM, it was clear that you should avoid using the username of “Administrator”. The username was found to be one of the most guessed. By disabling common usernames, you can avoid this risk. Password strength was also found to be important considering the large amount of log in attempts made overnight (less than 24 hours). By having a strong password, it adds another layer of defense.

