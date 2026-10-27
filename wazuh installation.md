# Wazuh Agent Installation on Windows 10

## Overview

This guide documents the installation and configuration of the Wazuh Agent on a Windows 10 virtual machine using VMware Workstation.

Wazuh is an open-source security monitoring platform used for threat detection, log analysis, file integrity monitoring, and security incident investigation.

**Lab Environment**

- Operating System: Windows 10
- Virtualisation: VMware Workstation
- Security Tool: Wazuh Agent
- Installation Method: Windows PowerShell
- Wazuh Agent Version: 4.14.9

## Step 1: Run PowerShell as Administrator

Open Windows PowerShell with administrator privileges.

To verify administrator access, execute:

```powershell
([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

Expected output:

```text
True
```

If PowerShell is not running as Administrator, execute:

```powershell
Start-Process powershell.exe -Verb RunAs
```

## Step 2: Download Wazuh Agent

Download the official Wazuh Agent MSI installer using PowerShell:

```powershell
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi" -OutFile "$env:TEMP\wazuh-agent.msi"
```

The installer will be downloaded to the Windows temporary directory.

## Step 3: Install Wazuh Agent

Install the Wazuh Agent using the following command:

```powershell
msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q WAZUH_MANAGER="YOUR_MANAGER_IP"
```

Replace `YOUR_MANAGER_IP` with the actual IP address of the Wazuh Manager.

**Note:** The Wazuh Manager must be deployed separately on a supported Linux system.

## Step 4: Configure Wazuh Agent

Open the Wazuh Agent configuration file:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

Locate the server configuration section:

```xml
<client>
  <server>
    <address>0.0.0.0</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

Replace `0.0.0.0` with the actual Wazuh Manager IP address.

Save the configuration file.

## Step 5: Start Wazuh Agent Service

Start the Wazuh Agent service:

```powershell
Start-Service WazuhSvc
```

Verify the service status:

```powershell
Get-Service WazuhSvc
```

Expected output:

```text
Status   Name       DisplayName
------   ----       -----------
Running  WazuhSvc   Wazuh
```

A `Running` status confirms that the Wazuh Agent service has started successfully.

## Step 6: Verify Wazuh Manager Connection

Check the agent's connection status:

```powershell
Select-String -Path "C:\Program Files (x86)\ossec-agent\wazuh-agent.state" -Pattern "^status"
```

Possible connection statuses:

| Status | Description |
|---|---|
| connected | Agent successfully connected to the Wazuh Manager |
| disconnected | Agent is not connected |
| pending | Agent is waiting for the Manager's acknowledgement |

## Step 7: Troubleshooting

During the installation, the Wazuh Agent service failed to remain running.

The following command was used to investigate the issue:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 40
```

The logs revealed the following errors:

```text
ERROR: (4112): Invalid server address found: '0.0.0.0'
ERROR: (1215): No client configured. Exiting.
```

**Root Cause**

The Wazuh Agent configuration contained the invalid Manager address `0.0.0.0`.

**Resolution**

1. Opened the `ossec.conf` configuration file.
2. Identified the invalid Manager address.
3. Configured the agent to use the correct Wazuh Manager address.
4. Restarted the Wazuh Agent service.
5. Verified that the Windows service was running.

## Conclusion

The Wazuh Agent was installed on the Windows 10 virtual machine, and the service was successfully started.

The next stage is to verify connectivity with the Wazuh Manager and confirm that Windows security events are being collected in the Wazuh Dashboard.

## References

- [Official Wazuh Documentation](https://documentation.wazuh.com/)
- [Installing Wazuh Agent on Windows](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
- [Wazuh Agent Troubleshooting](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/troubleshooting.html)
