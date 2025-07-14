# Azure Sentinel SOC Lab 🚨

This project demonstrates how to build a complete SOC lab environment using Microsoft Sentinel on Azure — from subscription creation to detecting malicious login attempts and visualizing attacks on a map.

---

## 🔐 Project Goals

- Simulate a SOC environment using Azure tools
- Collect logs from a virtual machine
- Forward logs to Microsoft Sentinel
- Detect failed login attempts
- Visualize attacker IPs on a world map

---

## 📦 Step-by-Step Setup

### 1. Azure Subscription (Using Student Email)
- Created a student Azure account using school email
- Got access to $100 in free credits and Azure for Students

### 🧑‍🎓 Don't have a student email?
See [`setup/student-alternative-subscription.md`](images/setup/student-alternative-subscription.md) for alternatives (e.g. free trial + credit card).

### 2. Resource Group and VNet
markdown
Copy
Edit
### 2. Resource Group and VNet

#### ✅ Creating a Resource Group

1. Go to the [Azure Portal](images/https://portal.azure.com).
2. In the search bar at the top, type `Resource groups`.  
   ![Search Resource Groups](images/resource-group-search.png)
3. Click the **Create** button.  
   ![Click Create](images/resource-group-create.png)
4. In the creation form:
   - Select your **Azure subscription**
   - Choose a **region**
   - Enter the **Resource Group name** (e.g., `RG-SOC-LAB`)  
   ![Enter Name](images/resource-group-name.png)
5. Click **Review + Create**, then finally click **Create**.

---

#### 🌐 Creating a Virtual Network

1. In the Azure Portal, search for `Virtual networks`.  
   ![Search Virtual Network](images/virtual-network-1.png)
2. Click **Create** and select the **existing resource group** `RG-SOC-LAB`.  
   ![Select Resource Group](images/virtual-network-2.png)
3. Enter a name for the VNet, e.g. `Vnet-soc-lab`, and complete the required fields.  
   ![Enter VNet Name](images/virtual-network-3.png)
4. Click **Review + Create**, then **Create** to finish deploying the virtual network.

### 3. Virtual Machine (VM) Creation

### 3. Virtual Machine (VM) Creation

#### 🖥️ Creating the Virtual Machine

1. In the Azure Portal, search for `Virtual machines`.  
   ![Search Virtual Machine](images/VM-1.png)
2. Click **Create**, and choose the **existing resource group** `RG-SOC-LAB`.  
   ![Create VM](images/VM-2.png)
3. In the Basics tab, fill in the following details:  
   - **VM Name:** `CORP-NET-EAST-1`  
   - **Image:** *Windows 10 Pro N, version 22H2 - x64 Gen2*  
   - **Username:** `labuser`  
   - **Password:** your choice  
   - ✅ Check **“I confirm I have an eligible Windows 10/11 license...”**
4. Click through to the other tabs:
   - **Disks:** keep default
   - **Networking:** select `Vnet-soc-lab`  
     ✅ Check **Delete public IP and NIC when VM is deleted**
   - All other settings: leave as default  
   ![VM Creation Confirmation](images/VM-3.png)
5. Click **Review + Create**, then **Create**.

---

### 4. Configure Firewall (NSG)

Once the deployment is complete:

1. Go to `Resource groups` → Select `RG-SOC-LAB`
2. Click on **`CORP-NET-EAST-1-nsg`**  
   ![NSG Selection](images/firewall-setting-2.png)
3. Delete the default **RDP rule** by clicking the trash icon.  
   ![Delete Default RDP Rule](images/firewall-setting-3.png)
4. Click on **Inbound security rules** → **Add rule**
5. In the new rule:
   - **Source:** Any
   - **Destination:** Any
   - **Port:** Any
   - **Protocol:** Any
   - **Action:** Allow
   - **Priority:** 100
   - **Name:** AllowAll
   - Description: Honeypot Lab  
   ![Add Firewall Rule](images/firewall-setting-4.png)
6. Click **Add** to save the rule.

---

### 5. Connect to the VM from Host Machine

#### 💻 Remote Desktop Access

1. In the Azure Portal, search for `Virtual machines`
2. Select the VM `CORP-NET-EAST-1`  
   ![Select VM](images/Test-1.png)
3. Copy the **Public IP address**  
   ![Copy Public IP](images/Test-2.png)
4. On your **host machine**, open the **Remote Desktop Connection** tool.
5. Paste the VM's Public IP address.  
   ![Enter IP in RDP](images/HostConnection-1.png)
6. Enter the credentials:
   - **Username:** `labuser`
   - **Password:** the one you set
7. Click **Connect**, then **Yes** to accept the certificate prompt.  
   ![RDP Connection Confirmation](images/HostConnection-2.png)

---

### 6. Disable Windows Firewall in the VM

Once inside the VM via RDP:

1. Open the **Start menu** and search for **"Firewall & network protection"**  
   ![Firewall Settings](images/HostConnection-3.png)
2. Turn off all three profiles:
   - **Domain network**
   - **Private network**
   - **Public network**  
   ![Disable All Profiles](images/HostConnection-4.png)

---

✅ Now, from your **host machine**, you should be able to ping the VM successfully, as the firewall is disabled and NSG allows all traffic.

---



### 7. Log Collection – Simulate Failed Login (Brute Force)

To simulate a brute-force login attempt and verify log generation in Windows Event Viewer:

#### 🔐 Simulate a Failed RDP Login

1. Open a **new RDP session** from your host machine.
2. Enter an invalid username (e.g., `bruteforce`) and any password.
   ![Invalid Username Attempt](images/BruteForceActivity-1.png)
3. Click **OK** — the authentication will fail.
   ![Failed Login](images/BruteForceActivity-2.png)

---

#### 🧾 Check Logs in Event Viewer (Inside the VM)

1. Inside the VM, open the **Start menu** and search for `Event Viewer`.
2. Navigate to:  
   `Windows Logs` → `Security`
3. Look for an event with **Event ID 4625** (failed login).
   ![Event Viewer Open](images/BruteForceActivity-3.png)
4. Click on the event to view the **username** that attempted the login (e.g., `bruteforce`).  
   ![Failed Login Details](images/BruteForceActivity-4.png)

✅ This confirms that failed login attempts are being recorded by the Windows Event Log, and will be collected once connected to Microsoft Sentinel.


### 8. Log Analytics Workspace & Microsoft Sentinel Setup

#### 📊 Create a Log Analytics Workspace

1. In the Azure Portal, search for `Log Analytics workspaces`.
2. Click **Create** to launch the workspace wizard.  
   ![Create LAW](images/Log-Analytics-1.png)
3. Fill in the required fields:
   - **Name:** `LAW-soc-lab-0001`
   - **Resource Group:** `RG-SOC-LAB`  
   ![Enter LAW Details](images/Log-Analytics-2.png)
4. Click **Review + Create**, then **Create** to deploy the workspace.

---

### 9. Connect Sentinel & Install Security Event Connector

#### 🧠 Add Microsoft Sentinel to LAW

1. In the Azure Portal, search for `Microsoft Sentinel`.
2. You should see your workspace (`LAW-soc-lab-0001`) listed. Click **Add** to connect Sentinel to the workspace.  
   ![Connect Sentinel](images/Log-Analytics-3.png)

---

#### 📦 Install Windows Security Events Content Pack

1. In the Sentinel workspace (`LAW-soc-lab-0001`), go to **Content Management** → **Content Hub**.
2. Search for `Security Events` and install the **Windows Security Events** pack.  
   ![Install Security Events](images/Log-Analytics-4.png)
3. Once installation finishes, click **Manage**.  
   ![Click Manage](images/Log-Analytics-5.png)

---

#### ⚙️ Configure Data Collection (via AMA)

1. In the content connector page, choose **Windows Security Events via AMA**.
2. Click **Open connector page**.  
   ![Open Connector Page](images/Log-Analytics-6.png)
3. Click **+ Create data collection rule** and fill in:
   - **Name:** `DCR-windows`
   - **Resources:** select your VM (`CORP-NET-EAST-1`)
   - **Collect:** choose **All Security Events**
4. Click **Review + Create**, then **Create**.  
   ![Create DCR](images/Log-Analytics-7.png)

---

### 10. Querying Failed Login Logs in Sentinel

1. Open **Log Analytics workspace** → select `LAW-soc-lab-0001`
2. Click **Logs**
3. Use the KQL query below to see failed login attempts:

```kql
SecurityEvent
| where EventID == 4625
| order by TimeGenerated desc


### 7. GeoIP Watchlist
- Uploaded IP database to Watchlist (name: `geoip`)

### 8. Query & Map Visualization

**KQL Query:** [`queries/geoip-attack-map.kql`](images/queries/geoip-attack-map.kql)

**Attack Map:** See [`screenshots/attack-map.png`](images/screenshots/attack-map.png)

```kusto
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "92.63.197.9"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
| project TimeGenerated, Computer, Attacker_IP = IpAddress, cityname, countryname, latitude, longitude
