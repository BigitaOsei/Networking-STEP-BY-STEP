# Verification Commands

## On the Switch (Core-SW)
- `show vlan brief`
- `show interfaces trunk`
- `show ip interface brief`
- `show running-config`

## On the Router (Border-RTR)
- `show ip interface brief`
- `show vlans`
- `show running-config`

## From End Devices (Laptops/PCs)
- Ping its own gateway (example: Sales PC → `ping 192.168.10.1`)
- Ping a device in another VLAN (example: Sales → Engineering)
- Ping the Switch management IP: `ping 192.168.99.2`
