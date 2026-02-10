
---

# Wazuh Active Response: Automated Brute Force Mitigation

This repository demonstrates the **Active Response** capabilities of the **Wazuh XDR** platform. The objective is to automatically mitigate security incidents based on predefined detection rules.

In this implementation, we simulate an **SSH brute-force attack** and configure Wazuh to automatically block the attacker’s IP address using a built-in `firewall-drop` active response script.

---

## Overview

* **Platform:** Wazuh XDR
* **Use Case:** SSH brute-force mitigation
* **Response Action:** Firewall IP block
* **Automation:** Yes (Active Response)
* **Test Method:** Simulated attack via `nmap`

---

## 1. Configuration: Defining the Active Response Command

First, ensure the `firewall-drop` command is defined in the Wazuh Manager configuration file (`ossec.conf`). This command uses default Wazuh scripts to interact with the host firewall.

```xml
<command>
  <name>firewall-drop</name>
  <executable>firewall-drop</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>
```

### Configuration Breakdown

* **Name**
  Identifier referenced by the Active Response configuration.

* **Executable**
  Script executed when the response is triggered.
  Location:

  ```
  /var/ossec/active-response/bin/
  ```

* **Timeout Allowed**
  Enables automatic removal of the firewall rule after a defined duration.

---

## 2. Implementation: Active Response Rule

Next, link the command to a specific security event. Add the following configuration to `/var/ossec/etc/ossec.conf`:

```xml
<ossec_config>
  <active-response>
    <disabled>no</disabled>
    <command>firewall-drop</command>
    <location>local</location>
    <rules_id>5763</rules_id>
    <timeout>180</timeout>
  </active-response>
</ossec_config>
```

### Rule Details

* **Rule ID:** `5763`
  Detects SSH brute-force authentication attempts.

* **Action:**
  Blocks the source IP using the system firewall.

* **Timeout:**
  The IP remains blocked for **180 seconds** before automatic removal.

> **Note:** This response executes locally on the Wazuh Manager.

---

## 3. Attack Simulation

To validate the configuration, a brute-force SSH attack is simulated from a remote host using `nmap`.

```bash
nmap --script ssh-brute \
     --script-args userdb=users.txt,passdb=passwords.txt \
     -p 22 <target_ip>
```

This command attempts multiple SSH login combinations using custom username and password lists.

---

## 4. Results & Verification

As the attack progresses, Wazuh monitors authentication logs in real time.

### Observed Behavior

* **SIEM Dashboard**

  * Thousands of `Authentication failed` alerts appear.
  * **Rule 651** confirms the Active Response module successfully executed the firewall block.

* **Network Connectivity**

  * All traffic from the attacker’s IP is dropped.
  * ICMP (ping) requests fail immediately after the rule is triggered.

* **Firewall State**

  * The malicious IP is added to the `iptables` DROP chain.
  * After the 180-second timeout expires, Wazuh automatically removes the rule and restores connectivity.

---

## Conclusion

This repository demonstrates how Wazuh Active Response can be used to:

* Detect SSH brute-force attacks
* Automatically block malicious IP addresses
* Enforce time-based firewall rules
* Reduce response time with no manual intervention

This approach significantly improves incident response efficiency and minimizes exposure during active attacks.

---
