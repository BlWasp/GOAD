# exchange

!!! success "Thanks!"
    Credits and huge thanks to [aleemladha](https://github.com/aleemladha) for his exchange role and his help to test the extension.

- Extension name : `exchange`
- Compatibility  : GOAD, GOAD-Light, FULL-GOAD-SCCM
- Providers : virtualbox/azure/vmware/aws/ludus/proxmox
- Add a machine  : srv01 (the-eyrie.sevenkingdoms.local)  (ip_range.21)

!!! warning "resources"
    Exchange is really HUGE, it will add a vm with at least 12Gb of RAM be sure your computer support it before install

!!! warning "impacts"
    Modify the ad schema and add a computer (warning the exchange machine is really heavy)

## Prerequisites

- GOAD, GOAD-Light or FULL-GOAD-SCCM installation

## Installation

- select your instance
```
load <instance_id>
```

- install the exchange extension
```
install_extension exchange
```
