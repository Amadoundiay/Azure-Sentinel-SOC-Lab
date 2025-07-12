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
See [`setup/student-alternative-subscription.md`](setup/student-alternative-subscription.md) for alternatives (e.g. free trial + credit card).

### 2. Resource Group and VNet
To create a Resource Group:

1. Go to the Azure Portal.
2. In the search bar, type `Resource groups`.  
   ![Search Resource Groups](screenshots/resource-group-search.png)
3. Click **Create**.  
   ![Click Create](screenshots/resource-group-create.png)
4. Enter your **Resource Group name**, for example: `RG-SOC-LAB`.  
   ![Enter Name](screenshots/resource-group-name.png)

---

After that, create a Virtual Network:

- Virtual Network Name: `Vnet-soc-lab`
- Associate it with the created Resource Group `RG-SOC-LAB`

### 3. Virtual Machine Setup
- Created a Windows VM
- Attached VM to VNet and NSG
- Allowed RDP, SSH, MySQL traffic (lab purpose)
- Connected to VM from host using RDP

### 4. Log Collection
- Triggered a failed login (EventID 4625)
- Verified via Event Viewer → Security logs

### 5. Log Analytics Workspace & Sentinel
- Created: `LAW-soc-lab-0000`
- Connected Sentinel to LAW
- Installed "Security Events" content

### 6. Forwarding VM Logs to Sentinel
- Created a Data Collection Rule (DCR)
- Enabled forwarding Windows security logs

### 7. GeoIP Watchlist
- Uploaded IP database to Watchlist (name: `geoip`)

### 8. Query & Map Visualization

**KQL Query:** [`queries/geoip-attack-map.kql`](queries/geoip-attack-map.kql)

**Attack Map:** See [`screenshots/attack-map.png`](screenshots/attack-map.png)

```kusto
let GeoIPDB_FULL = _GetWatchlist("geoip");
let WindowsEvents = SecurityEvent
    | where IpAddress == "92.63.197.9"
    | where EventID == 4625
    | order by TimeGenerated desc
    | evaluate ipv4_lookup(GeoIPDB_FULL, IpAddress, network);
WindowsEvents
| project TimeGenerated, Computer, Attacker_IP = IpAddress, cityname, countryname, latitude, longitude
