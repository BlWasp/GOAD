# FULL-GOAD-SCCM

Merge of the [GOAD](GOAD.md) lab and the [SCCM](SCCM.md) lab in one big environment :
the 3 GOAD domains (2 forests) + the `sccm.lab` forest with Microsoft Configuration Manager (MECM).

- GOAD part is identical to the GOAD lab (same users, groups, ACLs, vulnerabilities and scenarios)
- SCCM part is identical to the SCCM lab (same users, site configuration, NAA, client push, PXE...)
- A bidirectional forest trust is added between `sccm.lab` and `sevenkingdoms.local`
- Extensions (exchange, ws01, lx01, elk, wazuh, guacamole) stay optional and can be added on demand

## Servers
9 virtual machines :

| Hostname     | Domain                     | IP (last octet) | OS                  | Role                                     |
|--------------|----------------------------|-----------------|---------------------|------------------------------------------|
| kingslanding | sevenkingdoms.local        | .10             | Windows Server 2019 | DC (forest root), ADCS                   |
| winterfell   | north.sevenkingdoms.local  | .11             | Windows Server 2019 | DC (child domain)                        |
| meereen      | essos.local                | .12             | Windows Server 2016 | DC, LAPS                                 |
| castelblack  | north.sevenkingdoms.local  | .22             | Windows Server 2019 | IIS, MSSQL, WebDAV (defender disabled)   |
| braavos      | essos.local                | .23             | Windows Server 2016 | ADCS, MSSQL, WebDAV, LAPS                |
| DC           | sccm.lab                   | .40             | Windows Server 2019 | DC, DHCP (for PXE)                       |
| MECM         | sccm.lab                   | .41             | Windows Server 2019 | MECM primary site server (site P01)      |
| MSSQL        | sccm.lab                   | .42             | Windows Server 2019 | MECM database (MSSQL 2022)               |
| CLIENT       | sccm.lab                   | .43             | Windows Server 2019 | MECM client                              |

## Trusts

- `sevenkingdoms.local` <-> `essos.local` (bidirectional forest trust, same as GOAD)
- `sccm.lab` <-> `sevenkingdoms.local` (bidirectional forest trust)

## Providers

- vmware_esxi, proxmox, ludus

## Prerequisites

- The prerequisites are the same as the GOAD and SCCM labs for the chosen provider
- Resources : ~36GB of RAM for the 9 vms (vmware_esxi : the template provider set 4 vcpus/6GB on each vm, so ~54GB)
- Disk : the SCCM part alone takes ~100GB, plan the GOAD part on top of it
- Installation time : GOAD installation time + ~2,5 hours for the SCCM part
- The IPs `.21`, `.31`, `.32`, `.50`, `.51`, `.52` are kept free for the extensions, the sccm.lab DHCP distributes `.150` - `.200`

## Installation

```bash
python3 goad.py -t install -l FULL-GOAD-SCCM -p proxmox
```

## Writeups

- GOAD part : [https://mayfly277.github.io/categories/ad/](https://mayfly277.github.io/categories/ad/)
- SCCM part : [https://mayfly277.github.io/categories/sccm/](https://mayfly277.github.io/categories/sccm/)
