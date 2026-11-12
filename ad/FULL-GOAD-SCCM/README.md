# FULL-GOAD-SCCM

Merge of the [GOAD](../GOAD/README.md) lab and the [SCCM](../SCCM/README.md) lab in one big environment :
the 3 GOAD domains (2 forests) + the `sccm.lab` forest with Microsoft Configuration Manager (MECM).

- GOAD part is identical to the GOAD lab (same users, groups, ACLs, vulnerabilities and scenarios)
- SCCM part is identical to the SCCM lab (same users, site configuration, NAA, client push, PXE...)
- A bidirectional forest trust is added between `sccm.lab` and `sevenkingdoms.local`
- Extensions (exchange, ws01, lx01, elk, wazuh, guacamole) stay optional and can be added on demand

## Servers
This lab is composed of nine virtual machines:

| Inventory key | Hostname     | Domain                     | IP (last octet) | OS                  | Role                                     |
|---------------|--------------|----------------------------|-----------------|---------------------|------------------------------------------|
| dc01          | kingslanding | sevenkingdoms.local        | .10             | Windows Server 2019 | DC (forest root), ADCS                   |
| dc02          | winterfell   | north.sevenkingdoms.local  | .11             | Windows Server 2019 | DC (child domain)                        |
| dc03          | meereen      | essos.local                | .12             | Windows Server 2016 | DC, LAPS                                 |
| srv02         | castelblack  | north.sevenkingdoms.local  | .22             | Windows Server 2019 | IIS, MSSQL, WebDAV (defender disabled)   |
| srv03         | braavos      | essos.local                | .23             | Windows Server 2016 | ADCS, MSSQL, WebDAV, LAPS                |
| dc04          | DC           | sccm.lab                   | .40             | Windows Server 2019 | DC, DHCP (for PXE)                       |
| srv04         | MECM         | sccm.lab                   | .41             | Windows Server 2019 | MECM primary site server (site P01)      |
| srv05         | MSSQL        | sccm.lab                   | .42             | Windows Server 2019 | MECM database (MSSQL 2022)               |
| ws02          | CLIENT       | sccm.lab                   | .43             | Windows Server 2019 | MECM client                              |

## Forests, domains and trusts

- Forest `sevenkingdoms.local`
    - `sevenkingdoms.local` : kingslanding
    - `north.sevenkingdoms.local` (child domain) : winterfell, castelblack
- Forest `essos.local` : meereen, braavos
- Forest `sccm.lab` : DC, MECM, MSSQL, CLIENT

Trusts :

- `sevenkingdoms.local` <-> `essos.local` (bidirectional forest trust, same as GOAD)
- `sccm.lab` <-> `sevenkingdoms.local` (bidirectional forest trust)

## Network

- All the machines are in the same /24 network (`{{ip_range}}.0/24`)
- The IPs `.21`, `.31`, `.32`, `.50`, `.51`, `.52` are kept free for the extensions
- The DHCP server installed on the sccm.lab DC (for the MECM PXE scenario) distributes the range `.150` - `.200`

## Providers

- vmware_esxi
- proxmox
- ludus

## Prerequisites

- The prerequisites are the same as the GOAD and SCCM labs for the chosen provider
- Resources : ~36GB of RAM for the 9 vms (vmware_esxi : the template provider set 4 vcpus/6GB on each vm, so ~54GB)
- Disk : the SCCM part alone takes ~100GB, plan the GOAD part on top of it
- Installation time : GOAD installation time + ~2,5 hours for the SCCM part
- The lab download multiple files during the install (windows iso, mecm installation package, mssql installation package, ...), be sure to have a good internet connection

## Installation

```bash
python3 goad.py -t install -l FULL-GOAD-SCCM -p proxmox
python3 goad.py -t install -l FULL-GOAD-SCCM -p vmware_esxi
python3 goad.py -t install -l FULL-GOAD-SCCM -p ludus
```

- The playbook order is defined in `playbooks.yml` (key `FULL-GOAD-SCCM`) : GOAD playbooks, then the SCCM install/config playbooks
- If something goes wrong, you can replay the provisioning (ansible) from the goad console on your instance :

```bash
python3 goad.py
load <instance_id>
provision_lab                              # replay all the playbooks
provision_lab_from sccm-install.yml        # replay from a given playbook (ex: only the SCCM part)
```

## Users/Groups and associated vulnerabilites/scenarios

- GOAD part : see the [GOAD lab](../GOAD/README.md) and [https://mayfly277.github.io/categories/ad/](https://mayfly277.github.io/categories/ad/)
- SCCM part : see the [SCCM lab](../SCCM/README.md) and [https://mayfly277.github.io/categories/sccm/](https://mayfly277.github.io/categories/sccm/)
