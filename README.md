# 🛡️ Isolated Pentest Lab Setup — VirtualBox + Kali Linux (NAT Network)

![Status](https://img.shields.io/badge/status-complete-brightgreen)
![Platform](https://img.shields.io/badge/platform-VirtualBox-blue)
![OS](https://img.shields.io/badge/guest-Kali%20Linux%202026.2-informational)
![Author](https://img.shields.io/badge/author-MAliXCS-black)

A documented, reproducible lab setup for building an **isolated, internet-capable Kali Linux environment** using VirtualBox's **NAT Network** feature — completed as part of a cybersecurity internship lab task, with full screenshot evidence for every step.

📄 Full formal write-up: [`Cybersecurity_Lab_Report.docx`](./Cybersecurity_Lab_Report.docx)

---

## 📖 Overview

This repo documents the process of configuring a Kali Linux VM that is:
- **Isolated** from the host machine's physical LAN
- **Still able to reach the internet** for updates, tool installs, and recon practice
- Configured with a **static IP** for predictable, repeatable lab addressing

It also documents a real troubleshooting issue encountered during setup — an adapter reporting "not connected" after IP configuration — and how it was diagnosed and resolved.

---

## 🗺️ Network Architecture

```mermaid
graph LR
    Host[Host Machine] -->|NAT Network Gateway 10.0.0.1| VBoxNet[VirtualBox NAT Network<br/>10.0.0.0/24]
    VBoxNet --> Kali[Kali Linux VM<br/>eth0: 10.0.0.4/24<br/>DNS: 8.8.8.8]
    VBoxNet -->|NAT| Internet((Internet))
```

| Parameter | Value |
|---|---|
| Guest OS | Kali Linux 2026.2 (amd64) |
| Network Mode | NAT Network (`NatNetwork`) |
| Subnet | `10.0.0.0/24` |
| Kali IP | `10.0.0.4/24` |
| Gateway | `10.0.0.1` |
| DNS | `8.8.8.8` |
| Promiscuous Mode | Allow All |

---

## ⚙️ Prerequisites

- [Oracle VirtualBox](https://www.virtualbox.org/) installed on host
- [Kali Linux VirtualBox image](https://www.kali.org/get-kali/) (pre-built appliance)
- Host machine with virtualization (VT-x/AMD-V) enabled in BIOS/UEFI

---

## 🚀 Setup Steps

### Step 1 — Open VirtualBox's global network settings
`File → Tools → Network` inside VirtualBox Manager.

![Step 1](./Snaps/01-open-virtualbox-network-settings.png)

### Step 2 — Create the NAT Network
Under **NAT Networks**, create a network (`NatNetwork`) with IPv4 Prefix `10.0.0.0/24` and DHCP enabled at the network level.

![Step 2](./Snaps/02-configure-natnetwork-subnet.png)

### Step 3 — Import the Kali VM
From the VirtualBox Manager home screen, click **Open** to import a pre-built Kali appliance.

![Step 3](./Snaps/03-import-kali-vm-open.png)

### Step 4 — Select the extracted Kali VM folder

![Step 4](./Snaps/04-select-kali-vm-file.png)

### Step 5 — Attach Adapter 1 to the NAT Network
`Kali VM → Settings → Network → Adapter 1 → Attached to: NAT Network → NatNetwork`

![Step 5](./Snaps/05-attach-adapter1-to-natnetwork.png)

### Step 6 — Set Promiscuous Mode and confirm the virtual cable is connected
This checkbox controls the *simulated physical link* of the virtual NIC — see the [Troubleshooting Log](#-troubleshooting-log) below for why this matters.

![Step 6](./Snaps/06-adapter-cable-connected-promiscuous.png)

### Step 7 — Start the Kali VM
Confirm Adapter 1 shows `NAT Network, 'NatNetwork'` before booting.

![Step 7](./Snaps/07-start-kali-vm.png)

### Step 8 — Inside Kali, open the network connection editor
Click the network icon → **Edit Connections…**

![Step 8](./Snaps/08-open-edit-connections-in-kali.png)

### Step 9 — Set a static IP, gateway, and DNS
Under **IPv4 Settings**, set Method to **Manual**:
```
Address : 10.0.0.4
Netmask : 24
Gateway : 10.0.0.1
DNS     : 8.8.8.8
```

![Step 9](./Snaps/09-set-static-ip-gateway-dns.png)

### Step 10 — Verify the interface
```bash
ip a
```

![Step 10](./Snaps/10-verify-ip-a.png)

### Step 11 — Verify internet connectivity (ICMP)
```bash
ping google.com
```

![Step 11](./Snaps/11-ping-google-success.png)

### Step 12 — Verify application-layer connectivity (browser)
Opened Firefox and loaded `fortinet.com` to confirm HTTP/HTTPS traffic, not just ICMP.

![Step 12](./Snaps/12-browser-fortinet-success.png)

Snapshot taken immediately after this working state to preserve a known-good baseline.

---

## 🐛 Troubleshooting Log

### Issue: Network adapter shows "not connected" after applying static IP

**Symptom:** Static IP, gateway, and DNS were all configured correctly, but the interface reported no link / not connected, and all connectivity failed.

**Root Cause:** VirtualBox's per-adapter **"Cable Connected"** setting was unchecked. This operates at the *virtual hardware* level — below the guest OS's network stack — and simulates an unplugged Ethernet cable regardless of how correctly the OS itself is configured.

**Fix:**
1. Shut down the Kali VM.
2. `Settings → Network → Adapter 1`.
3. Check **"Virtual Cable Connected"** (see Step 6 screenshot above).
4. Start the VM.

**Lesson:** When network config looks correct but nothing connects, check the hypervisor's virtual hardware settings before re-checking the OS config — the fault may be a layer below where you're looking.

---

## ✅ Verification Results

| Test | Command / Action | Result |
|---|---|---|
| Interface state | `ip a` | ✅ eth0 `state UP`, `inet 10.0.0.4/24` |
| ICMP reachability | `ping google.com` | ✅ Replies received, 0% loss |
| DNS resolution | Implicit in ping | ✅ Resolved via `8.8.8.8` |
| HTTP/HTTPS traffic | Browser → fortinet.com | ✅ Page loaded fully |
| Environment checkpoint | VirtualBox Snapshot | ✅ Captured |

---

## 📌 Notes / Next Steps

- [ ] Add a second isolated VM (e.g. Metasploitable2) to the same NAT Network for attack/defense practice
- [ ] Revisit Promiscuous Mode (`Allow All`) as a deliberate choice before multi-VM sniffing exercises
- [ ] Document firewall rules once additional VMs are added
- [ ] Add a Host-Only adapter for a management channel isolated from internet-facing traffic

---

## 📁 Repo Structure

```
.
├── README.md
├── Cybersecurity_Lab_Report.docx     # Full formal write-up
└── screenshots/
    ├── 01-open-virtualbox-network-settings.png
    ├── 02-configure-natnetwork-subnet.png
    ├── 03-import-kali-vm-open.png
    ├── 04-select-kali-vm-file.png
    ├── 05-attach-adapter1-to-natnetwork.png
    ├── 06-adapter-cable-connected-promiscuous.png
    ├── 07-start-kali-vm.png
    ├── 08-open-edit-connections-in-kali.png
    ├── 09-set-static-ip-gateway-dns.png
    ├── 10-verify-ip-a.png
    ├── 11-ping-google-success.png
    ├── 12-browser-fortinet-success.png
    ├── step_1_add_new_vm_mechine_click_on_new.png
    ├── step_2_name_the_machine__choose_the_folder_where_you_want_to_store_this_vm__adn_give_the_path_of_iso_of_your_windows_vm.png
    ├── step_3_set_the_password_and_naem___set_host_name_and__finish.png
    ├── step_4_windows_10_vm_is_nowinstalled_completly__not_shut_it_down_completly.png
    ├── step_5_click_on_network.png
    ├── step_6_in_adapter_1_elect_natnetwork__select_allow_all_and_check_the_virtual_cable_connected_if_it_is_unchecked.png
    ├── step_7_start_the_win_10_vm.png
    ├── step_8_open_control_panel_in_win_10_vm_and_open_network_and_sharing_center.png
    ├── step_9_click_on_change_adapter_setting_on_left.png
    ├── step_10_right_click_on_adapter_and_clicl_on_propertes.png
    ├── step_11_select_1p_v4_and_click_on_properties.png
    ├── step_12_set_ip__subnetmask__gateway_and_dns_and_click_ok_and_then_apply_and_ok.png
    ├── step_13_ping_own_ip__gateway__dns__google_adn_kali_vm.png
    ├── step_A_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_click_on_start_button_and_search_for_run_or_u_can_open_it_by_pressing_widows___r_on_keyboard.png
    ├── Step_B_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_Search_wf_msc_and_hit_enter.png
    ├── Step_C_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_it_will_open_the_firewall_advance_security_setting_on_left_click_on_inbound_rules.png
    ├── Step_D_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm__scrcoll_down_and_look_for_both_file_and_print_sharing__eco_icmp_pv__and_enable_them.png
    └── Step_E_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm__NOW_WE_CAN_SEE_ping_i_successfull_for_kali_to_windows_10_vm.png
```

---

## 🖥️ Adding a Windows 10 Target VM

A second VM was added to the lab to act as a **target host** for reconnaissance and exploitation exercises, built with VirtualBox's unattended-installation feature and attached to the same isolated `NatNetwork` as the Kali VM, then given a static IP so both hosts have predictable addressing.

### Step 1 — Create a new VM
In VirtualBox Manager, click **New**.

![Step 1](./Snaps/step_1_add_new_vm_mechine_click_on_new.png)

### Step 2 — Name the VM and point to the Windows ISO
Set the **VM Name**, choose the **VM Folder**, and point **ISO Image** to the Windows 10 ISO. Leave **Proceed with Unattended Installation** checked.

![Step 2](./Snaps/step_2_name_the_machine__choose_the_folder_where_you_want_to_store_this_vm__adn_give_the_path_of_iso_of_your_windows_vm.png)

### Step 3 — Set credentials, host name, and finish
Under **Set up unattended guest OS installation**, set the **User Name**, **Password**, and **Host Name**, then click **Finish**.

![Step 3](./Snaps/step_3_set_the_password_and_naem___set_host_name_and__finish.png)

### Step 4 — Let the install complete, then shut down
Allow the unattended installation to finish. Once Windows 10 boots to the desktop, shut the VM down completely before changing network settings.

![Step 4](./Snaps/step_4_windows_10_vm_is_nowinstalled_completly__not_shut_it_down_completly.png)

### Step 5 — Open the VM's Network settings
With the VM powered off, select it in VirtualBox Manager and open **Settings → Network**.

![Step 5](./Snaps/step_5_click_on_network.png)

### Step 6 — Attach Adapter 1 to the NAT Network
Set **Adapter 1 → Attached to: NAT Network → NatNetwork**, set **Promiscuous Mode** to **Allow All**, and confirm **Virtual Cable Connected** is checked (this is the same setting responsible for the "not connected" issue documented in the [Troubleshooting Log](#-troubleshooting-log) above — check it here too).

![Step 6](./Snaps/step_6_in_adapter_1_elect_natnetwork__select_allow_all_and_check_the_virtual_cable_connected_if_it_is_unchecked.png)

### Step 7 — Start the Windows 10 VM
Confirm Adapter 1 shows **NAT Network, 'NatNetwork'**, then start the VM.

![Step 7](./Snaps/step_7_start_the_win_10_vm.png)

### Step 8 — Open Network and Sharing Center
Inside the Windows 10 VM, open **Control Panel → Network and Sharing Center**.

![Step 8](./Snaps/step_8_open_control_panel_in_win_10_vm_and_open_network_and_sharing_center.png)

### Step 9 — Change adapter settings
Click **Change adapter settings** in the left-hand pane.

![Step 9](./Snaps/step_9_click_on_change_adapter_setting_on_left.png)

### Step 10 — Open the adapter's Properties
Right-click the Ethernet adapter and select **Properties**.

![Step 10](./Snaps/step_10_right_click_on_adapter_and_clicl_on_propertes.png)

### Step 11 — Open IPv4 Properties
Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.

![Step 11](./Snaps/step_11_select_1p_v4_and_click_on_properties.png)

### Step 12 — Set a static IP, subnet, gateway, and DNS
Choose **Use the following IP address** and set:
```
IP address      : 10.0.0.10
Subnet mask     : 255.255.255.0
Default gateway : 10.0.0.1
Preferred DNS   : 8.8.8.8
```
Click **OK → Apply → OK**.

![Step 12](./Snaps/step_12_set_ip__subnetmask__gateway_and_dns_and_click_ok_and_then_apply_and_ok.png)

### Step 13 — Verify connectivity
Open Command Prompt and ping the VM's own IP, the gateway, the DNS server, Google, and the Kali VM to confirm the static configuration is working end-to-end.

![Step 13](./Snaps/step_13_ping_own_ip__gateway__dns__google_adn_kali_vm.png)

---

## 🔥 Troubleshooting: Kali → Windows 10 Ping Failure (ICMPv4 Firewall Rule)

**Symptom:** Windows 10 could ping the Kali VM without issue, but a ping from Kali to the Windows 10 target (`10.0.0.10`) got no reply. Both hosts could already reach the internet independently, so this pointed at the Windows side specifically.

**Root Cause:** This is standard, expected **Windows Defender Firewall** behavior — inbound ICMP Echo Requests are blocked by default on a fresh Windows install, so Kali's pings were being silently dropped at the Windows host rather than lost on the network.

**Fix:**

### Step A — Open Run
On the Windows 10 VM, click **Start** and search for **Run**, or press **Windows + R**.

![Step A](./Snaps/step_A_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_click_on_start_button_and_search_for_run_or_u_can_open_it_by_pressing_widows___r_on_keyboard.png)

### Step B — Launch the firewall console
In Run, type `wf.msc` and hit **Enter**.

![Step B](./Snaps/Step_B_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_Search_wf_msc_and_hit_enter.png)

### Step C — Open Inbound Rules
In **Windows Defender Firewall with Advanced Security**, click **Inbound Rules** on the left.

![Step C](./Snaps/Step_C_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm_it_will_open_the_firewall_advance_security_setting_on_left_click_on_inbound_rules.png)

### Step D — Enable the ICMPv4 Echo Request rules
Scroll down to **File and Printer Sharing (Echo Request – ICMPv4-In)** and enable it (both the Domain and Private/Public entries shown, as needed for the lab network profile).

![Step D](./Snaps/Step_D_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm__scrcoll_down_and_look_for_both_file_and_print_sharing__eco_icmp_pv__and_enable_them.png)

### Step E — Confirm the fix from Kali
From the Kali VM, re-run the ping against `10.0.0.10` (along with the gateway and Kali's own IP for comparison) to confirm the Windows 10 target now responds.

![Step E](./Snaps/Step_E_TO_SOLVE_ping_issue_in_kili_to_windows__in_win_10_vm__NOW_WE_CAN_SEE_ping_i_successfull_for_kali_to_windows_10_vm.png)

**Lesson:** Same theme as the earlier "Virtual Cable Connected" issue — when connectivity fails asymmetrically (one direction works, the other doesn't), check host-level filtering (firewalls, ICMP rules) before assuming a network or hypervisor misconfiguration.

---

## 🗺️ Extended Lab Architecture (Two-VM)

| VM / Role | IP Address (NatNetwork `10.0.0.0/24`) |
|---|---|
| NAT Network Gateway | `10.0.0.1` |
| Kali Linux (Attacker) | `10.0.0.4` — static, DNS `8.8.8.8` |
| Windows 10 (Target) | `10.0.0.10` — static, DNS `8.8.8.8` |

### Windows 10 VM Verification Results

| Test | Command / Action | Result |
|---|---|---|
| Static IP applied | `ipconfig` | ✅ `10.0.0.10/24`, gateway `10.0.0.1` |
| Self / gateway / DNS reachability | `ping 10.0.0.10`, `ping 8.8.8.8` | ✅ Replies received |
| Internet reachability | `ping google.com` | ✅ Replies received |
| Kali → Windows ICMP (before fix) | `ping 10.0.0.10` from Kali | ❌ No reply (blocked by Windows Firewall) |
| Kali → Windows ICMP (after fix) | `ping 10.0.0.10` from Kali | ✅ Replies received, 0% loss |

---

## 👤 Author

**M. Ali** — [@MAliXCS](https://github.com/MAliXCS)
Cybersecurity Intern at Network-Walks | Documenting hands-on lab work as I go

---

## 📄 License

This repository is provided for educational/documentation purposes. Feel free to fork and adapt for your own lab notes.
