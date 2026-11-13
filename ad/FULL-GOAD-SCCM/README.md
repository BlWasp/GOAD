# FULL-GOAD-SCCM

Merge of the [GOAD](../GOAD/README.md) lab and the [SCCM](../SCCM/README.md) lab in one big environment :
the 3 GOAD domains (2 forests) + the `sccm.lab` forest with Microsoft Configuration Manager (MECM).

- GOAD part is identical to the GOAD lab (same users, groups, ACLs, vulnerabilities and scenarios)
- SCCM part is identical to the SCCM lab (same users, site configuration, NAA, client push, PXE...)
- A bidirectional forest trust is added between `sccm.lab` and `sevenkingdoms.local`
- 5 cross-forest attack paths are added between `sccm.lab` and `sevenkingdoms.local` (see below)
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

## Cross-forest attack paths (sccm.lab <-> sevenkingdoms.local)

A forest trust applies SID filtering by default : classic intra-forest primitives (SID history,
golden/diamond ticket reuse) do not cross it. The paths below use what genuinely crosses a forest
trust : foreign security principal membership, ACLs granted to a foreign principal, shared secrets,
and application-level trust (SCCM managing clients across the trust).

**GOAD -> SCCM**

- **A. Kerberoast + password reuse** : `ravens_svc` (sevenkingdoms.local, SPN `HTTP/kingslanding.sevenkingdoms.local`)
  shares its password with `dave` (sccm.lab, member of `SCCM-Admins` -> CM console Full Administrator).
  Kerberoast `ravens_svc` on the GOAD side, crack it, reuse the password as `dave` on sccm.lab.
- **B. Foreign security principal in a domain-local group** : `sevenkingdoms.local\maester.pycelle` is a
  member of `sccm.lab`'s domain-local group `SCCM-Support-Access`, which is a local Administrator on MECM (srv04).

**SCCM -> GOAD**

- **C. SCCM push account with cross-forest local admin** : `sccm.lab\sccm-client-push` (the SCCM client
  push account) is a local Administrator on castelblack (srv02, north.sevenkingdoms.local) - realistic for
  a shared SCCM hierarchy managing machines in both forests.
- **D. Cross-forest ACL (DCSync)** : `sccm.lab\sccm-account-da` (already Domain Admin of sccm.lab) has been
  granted `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All` on `sevenkingdoms.local`'s domain
  root - a legacy over-permissioned "monitoring" grant. Taking over the SCCM site (or `sccm-account-da`
  directly) allows a DCSync of sevenkingdoms.local.
- **E. Coercion + NTLM relay to ADCS (ESC8)** : no extra data needed. sevenkingdoms.local's CA (dc01) has web
  enrollment enabled by default and its firewall disabled. From a foothold in sccm.lab, coerce a sevenkingdoms
  host (PetitPotam/dfscoerce/printerbug) and relay the authentication to `http://kingslanding.sevenkingdoms.local/certsrv/`
  to obtain a certificate for the coerced account.

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
