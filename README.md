# Cisco ISE EAP-TLS Wireless Authentication Lab (Meraki MR36)

A lab on implementing **certificate-based wireless authentication (EAP-TLS)** using **Cisco ISE** as the RADIUS/CA server and a **Meraki MR36** access point, with dynamic VLAN assignment separating IT and HR wireless clients — no passwords involved, authentication is entirely certificate-driven.

---

## 📌 Project Objective

Design a secure enterprise wireless network where clients authenticate using **digital certificates (EAP-TLS)** instead of passwords, issued and validated by an internal Certificate Authority on Cisco ISE, with the RADIUS server dynamically assigning each client to the correct VLAN (IT → VLAN 10, HR → VLAN 20) based on their identity group — then prove the full flow works end-to-end for both departments.

---

## 🖧 Environment Overview

| Component | Details |
|---|---|
| Wireless AP | Cisco Meraki MR36 |
| Switch | Cisco Catalyst (trunking VLANs to the AP) |
| Edge Router | NAT + routing to the internet |
| RADIUS / CA Server | Cisco ISE (Identity Services Engine) — internal CA issuing EAP-TLS certificates |
| Hypervisor | VMware ESXi hosting the ISE VM |
| SSID | `LAB-EAPTLS` — WPA2-Enterprise, EAP-TLS only |
| IT Client | Certificate-authenticated → VLAN 10 |
| HR Client | Certificate-authenticated → VLAN 20 |

---

## ⚙️ Step 1 — Core Network Foundation

Before touching wireless, confirmed the underlying wired network was healthy: VLAN database and trunking on the Catalyst switch, edge router interface/routing status, and NAT translations to the internet.

![Catalyst Switch VLAN Database](images/01-Catalyst-Switch-VLAN-Database.png)
![Catalyst Trunks and DHCP Bindings](images/02-Catalyst-Trunks-and-DHCP-Bindings.png)
![Edge Router IP Interface Status](images/03-Edge-Router-IP-Interface-Status.png)
![Edge Router Routing Table](images/04-Edge-Router-Routing-Table.png)
![Edge Router NAT Statistics and Translations](images/05-Edge-Router-NAT-Statistics-and-Translations.png)

---

## ⚙️ Step 2 — Meraki Switch and AP Setup

Configured the Meraki MS130 trunk port carrying the wireless VLANs, verified the Layer 2 topology in the Meraki dashboard, and confirmed the MR36 access point was online.

![Meraki MS130 Trunk Port Configuration](images/06-Meraki-MS130-Trunk-Port-Configuration.png)
![Meraki Dashboard Layer2 Topology](images/07-Meraki-Dashboard-Layer2-Topology.png)
![Meraki MR36 Access Points Online](images/08-Meraki-MR36-Access-Points-Online.png)

---

## ⚙️ Step 3 — Deploying Cisco ISE on ESXi

Hosted the ISE virtual appliance on VMware ESXi, placed its management interface and dedicated ISE VLAN (VLAN 30) on separate port groups, and confirmed the VM's resource allocation and virtual NICs.

![ESXi Management VMkernel Interface](images/09-ESXi-Management-VMkernel-Interface.png)
![ESXi ISE VLAN30 Port Group](images/10-ESXi-ISE-VLAN30-Port-Group.png)
![ESXi Cisco ISE VM Resources and NICs](images/11-ESXi-Cisco-ISE-VM-Resources-and-NICs.png)

Confirmed ISE was up, licensed for the lab, and all core application services were running.

![Cisco ISE Version and Build](images/12-Cisco-ISE-Version-and-Build.png)
![Cisco ISE Application Services Status](images/13-Cisco-ISE-Application-Services-Status.png)
![Cisco ISE Dashboard Summary](images/14-Cisco-ISE-Dashboard-Summary.png)

---

## ⚙️ Step 4 — Certificate Infrastructure (Internal CA)

Reviewed ISE's system certificates and EAP server certificate, configured the internal CA settings, and confirmed the CA certificate hierarchy — the foundation that makes EAP-TLS possible without an external CA.

![Cisco ISE System Certificates](images/15-Cisco-ISE-System-Certificates.png)
![Cisco ISE EAP Server Certificate](images/16-Cisco-ISE-EAP-Server-Certificate.png)
![Cisco ISE Internal CA Settings](images/17-Cisco-ISE-Internal-CA-Settings.png)
![Cisco ISE CA Certificate Hierarchy](images/18-Cisco-ISE-CA-Certificate-Hierarchy.png)

Created a certificate template for issuing client certificates, and confirmed a Windows client's issued certificate chained correctly back to the internal CA.

![Cisco ISE EAPTLS Certificate Template](images/19-Cisco-ISE-EAPTLS-Certificate-Template.png)
![Windows EAPTLS Client Certificate Chain](images/20-Windows-EAPTLS-Client-Certificate-Chain.png)

---

## ⚙️ Step 5 — Network Device and Endpoint Groups

Registered the Meraki MR36 as a network device (RADIUS client) in ISE, and created an approved endpoint group for trusted IT devices.

![Cisco ISE MR36 Network Device Entry](images/21-Cisco-ISE-MR36-Network-Device-Entry.png)
![Cisco ISE Approved IT Endpoint Group](images/22-Cisco-ISE-Approved-IT-Endpoint-Group.png)

---

## ⚙️ Step 6 — EAP-TLS Policy Configuration

Built a certificate authentication profile, restricted allowed protocols to EAP-TLS only, and created the authentication and authorization policies — including separate authorization profiles pushing IT clients to VLAN 10 and HR clients to VLAN 20.

![Cisco ISE EAPTLS Certificate Authentication Profile](images/23-Cisco-ISE-EAPTLS-Certificate-Authentication-Profile.png)
![Cisco ISE EAPTLS Only Allowed Protocols](images/24-Cisco-ISE-EAPTLS-Only-Allowed-Protocols.png)
![Cisco ISE EAPTLS Authentication Policy](images/25-Cisco-ISE-EAPTLS-Authentication-Policy.png)
![Cisco ISE EAPTLS Authorization Policies](images/26-Cisco-ISE-EAPTLS-Authorization-Policies.png)
![Cisco ISE VLAN10 Authorization Profile](images/27-Cisco-ISE-VLAN10-Authorization-Profile.png)
![Cisco ISE VLAN20 Authorization Profile](images/28-Cisco-ISE-VLAN20-Authorization-Profile.png)

---

## ⚙️ Step 7 — Meraki Wireless (SSID) Configuration

Configured the `LAB-EAPTLS` SSID for WPA2-Enterprise security, pointed it at the ISE RADIUS servers, and set the SSID to bridge mode with VLAN override so ISE's returned VLAN assignment actually takes effect per client.

![Meraki LAB-EAPTLS SSID Enterprise Security](images/29-Meraki-LAB-EAPTLS-SSID-Enterprise-Security.png)
![Meraki LAB-EAPTLS WPA2 and Direct Access](images/30-Meraki-LAB-EAPTLS-WPA2-and-Direct-Access.png)
![Meraki LAB-EAPTLS RADIUS Servers](images/31-Meraki-LAB-EAPTLS-RADIUS-Servers.png)
![Meraki LAB-EAPTLS Bridge Mode and VLAN Override](images/32-Meraki-LAB-EAPTLS-Bridge-Mode-and-VLAN-Override.png)

---

## ✅ Step 8 — Client Configuration and Connection

Built a Windows wireless profile manually for the EAP-TLS SSID, selected the correct authentication method and computer authentication mode, accepted the certificate trust prompt, and confirmed a successful connection.

![Windows Wireless Profile Setup Start](images/33-Windows-Wireless-Profile-Setup-Start.png)
![Windows Manual Wireless Network Selection](images/34-Windows-Manual-Wireless-Network-Selection.png)
![Windows LAB-EAPTLS WPA2 Enterprise Profile](images/35-Windows-LAB-EAPTLS-WPA2-Enterprise-Profile.png)
![Windows LAB-EAPTLS Profile Created](images/36-Windows-LAB-EAPTLS-Profile-Created.png)
![Windows EAPTLS Authentication Method](images/37-Windows-EAPTLS-Authentication-Method.png)
![Windows EAPTLS Computer Authentication Mode](images/38-Windows-EAPTLS-Computer-Authentication-Mode.png)
![Windows LAB-EAPTLS Certificate Trust Prompt](images/39-Windows-LAB-EAPTLS-Certificate-Trust-Prompt.png)
![Windows LAB-EAPTLS Connected](images/40-Windows-LAB-EAPTLS-Connected.png)

---

## ✅ Step 9 — End-to-End Verification for Both Departments

Confirmed the IT client landed on VLAN 10 with the correct network details, then validated it from the CLI. Repeated the test for the HR client on VLAN 20.

![IT Client EAPTLS VLAN10 Network Details](images/41-IT-Client-EAPTLS-VLAN10-Network-Details.png)
![IT Client EAPTLS VLAN10 CLI Validation](images/42-IT-Client-EAPTLS-VLAN10-CLI-Validation.png)
![HR Client EAPTLS VLAN20 CLI Validation](images/43-HR-Client-EAPTLS-VLAN20-CLI-Validation.png)

### Final proof — ISE live logs for both clients
Reviewed ISE's RADIUS live logs showing both the IT and HR clients successfully authenticating via EAP-TLS and being placed into their correct VLANs by policy.

![Cisco ISE Live Logs IT and HR EAPTLS Results](images/44-Cisco-ISE-Live-Logs-IT-and-HR-EAPTLS-Results.png)

---

## ✅ Results

- Deployed **Cisco ISE** on VMware ESXi as a RADIUS server and internal Certificate Authority.
- Implemented **certificate-based wireless authentication (EAP-TLS)** on a Meraki MR36 access point — no passwords involved in the authentication exchange.
- Configured **dynamic VLAN assignment**, correctly placing IT clients on VLAN 10 and HR clients on VLAN 20 based on their certificate identity and ISE authorization policy.
- Restricted the SSID to **EAP-TLS only**, rejecting any client that isn't presenting a valid certificate issued by the internal CA.
- Verified the entire flow end-to-end — from certificate issuance to Windows client connection to ISE's live authentication logs — for both departments.

---

## 🛠️ Skills Demonstrated

`Cisco ISE` `EAP-TLS` `802.1X` `Certificate-Based Authentication` `Internal PKI / CA` `Cisco Meraki MR36` `Wireless Security (WPA2-Enterprise)` `Dynamic VLAN Assignment` `RADIUS` `VMware ESXi` `Network Troubleshooting`
