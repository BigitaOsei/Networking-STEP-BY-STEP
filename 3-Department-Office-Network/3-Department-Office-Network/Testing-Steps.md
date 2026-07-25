# Testing & Verification Steps

## 1. Check Router Interfaces
```cisco
Office-Router# show ip interface brief

Check VLANs on Switch
Office-Switch# show vlan brief

Check Trunk
ciscoOffice-Switch# show interfaces trunk

Connectivity Tests (from PCs)
Test,Command,Expected Result
IT Gateway,ping 192.168.10.1,Success
HR Gateway,ping 192.168.10.33,Success
MGMT Gateway,ping 192.168.10.65,Success
IT → HR,ping 192.168.10.40,Success
IT → MGMT,ping 192.168.10.70,Success
HR → MGMT,ping 192.168.10.70,Success


## 5. PC IP Configuration

| PC       | IP Address      | Subnet Mask       | Default Gateway  |
|----------|-----------------|-------------------|------------------|
| PC-IT    | 192.168.10.10   | 255.255.255.224   | 192.168.10.1     |
| PC-HR    | 192.168.10.40   | 255.255.255.224   | 192.168.10.33    |
| PC-MGMT  | 192.168.10.70   | 255.255.255.240   | 192.168.10.65    |
