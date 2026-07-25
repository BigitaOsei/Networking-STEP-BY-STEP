# IP Addressing Plan

## Base Network
192.168.10.0/24

## Subnet Calculations (VLSM)

### IT Department (30 hosts)
- Required: 32 addresses → /27
- Network: 192.168.10.0/27
- Mask: 255.255.255.224
- Usable: 192.168.10.1 – 192.168.10.30
- Broadcast: 192.168.10.31
- Gateway: 192.168.10.1

### HR Department (20 hosts)
- Required: 32 addresses → /27
- Network: 192.168.10.32/27
- Mask: 255.255.255.224
- Usable: 192.168.10.33 – 192.168.10.62
- Broadcast: 192.168.10.63
- Gateway: 192.168.10.33

### Management Department (10 hosts)
- Required: 16 addresses → /28
- Network: 192.168.10.64/28
- Mask: 255.255.255.240
- Usable: 192.168.10.65 – 192.168.10.78
- Broadcast: 192.168.10.79
- Gateway: 192.168.10.65
