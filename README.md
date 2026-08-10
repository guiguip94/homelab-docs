# 🚀 HomeLab TSSR — Infrastructure & Architecture Technique

Ce dépôt centralise la documentation technique, l'architecture réseau et les procédures d'administration de mon HomeLab. Déployé sous **Proxmox VE 8.x**, cet environnement constitue le support d'application pratique de ma formation de **Technicien Supérieur Systèmes et Réseaux (TSSR)**.

---

## 🖥️ Matériel Physique & Équipements Réseau

* **Serveur hôte principal :** NiPoGi AM06 Pro (AMD Ryzen 7 — 8 cœurs / 16 threads, 32 Go RAM DDR4)
* **Stockage :** SSD NVMe PCIe 4.0 1 To (OS & VMs) + SSD SATA 512 Go (Backups, ISOs & Partages)
* **Équipement Réseau Cœur :** Switch MokerLink Administrable Gigabit (Trunk 802.1Q vers Proxmox)
* **Extension Réseau :** Switch non administrable Gigabit (Déport de ports sur le VLAN 10)
* **Accès Internet & WAN :** Freebox (WAN1) + Partage de connexion 4G/5G Nomade (WAN2 Failover)

---

## 🌐 Plan d'Adressage Réseau & Segmentation VLANs

| VLAN / Zone | Nom | Plage IP / Subnet | Passerelle (pfSense) | Usage & Équipements Associés |
| :---: | :--- | :--- | :--- | :--- |
| **WAN1** | Box Internet (Freebox) | `DHCP / Dynamic` | — | Connexion principale (Mode Bridge / DMZ) |
| **VLAN 1** | Management & Corosync | `192.168.1.0/24` | `192.168.1.1` | Proxmox VE (`pve01`), switchs MokerLink, Cluster HA |
| **VLAN 10** | Perso & Média / IoT | `192.168.10.0/24` | `192.168.10.1` | Wi-Fi Perso, Smart TV, Sonos, Home Assistant, Jellyfin |
| **VLAN 20** | Lab TSSR / Services | `192.168.20.0/24` | `192.168.20.1` | Windows Server 2025 (AD DS, DNS, DHCP, Hyper-V Nested) |
| **VLAN 30** | Clients TSSR & VDI | `192.168.30.0/24` | `192.168.30.1` | VM Win 11 Pro (RDP Parents / VDI Mobile), Client Debian 12 |
| **VLAN 40** | VLAN Nomade (4G/5G) | `192.168.40.0/24` | `192.168.40.1` | Partage de connexion 4G/5G (Failover WAN2 / Secours) |
| **VLAN 50** | VLAN Invités (Guest) | `192.168.50.0/24` | `192.168.50.1` | Réseau Wi-Fi Invités totalement isolé + Accès QR Code |
| **VLAN 99** | DMZ | `192.168.99.0/24` | `192.168.99.1` | Services exposés & relais VDI Mobile |

---

## 📐 Schéma d'Architecture Réseau (Mermaid)

```mermaid
graph TD
    WAN1[🌐 WAN1 : Freebox] -->|Interface WAN| PFSENSE[🛡️ pfSense Firewall]
    MOBILE[📱 WAN2 : 4G/5G Nomade] -->|VLAN 40 / Failover| PFSENSE

    subgraph PROXMOX[🖥️ Proxmox VE 8.x - NiPoGi AM06 Pro Ryzen 7]
        PFSENSE

        subgraph VLAN1[🔧 VLAN 1 - Management]
            PVEUI[Proxmox Web UI / Corosync]
        end

        subgraph VLAN10[🏠 VLAN 10 - Perso & Média]
            HASS[🤖 Home Assistant]
            JELLY[🎬 Jellyfin]
            MEDIA[Smart TV / Sonos]
        end

        subgraph VLAN20[⚙️ VLAN 20 - Lab TSSR]
            WIN2025[🖥️ Win Server 2025 AD DS / Hyper-V]
        end

        subgraph VLAN30[👥 VLAN 30 - Clients & VDI]
            WIN11[💻 VM Win 11 VDI / RDP Parents]
            DEBIAN[🐧 Client Debian 12]
        end

        subgraph VLAN50[📲 VLAN 50 - Guest]
            GUEST[📶 Wi-Fi Invités / QR Code]
        end
    end

    PFSENSE -->|VLAN 1| VLAN1
    PFSENSE -->|VLAN 10| VLAN10
    PFSENSE -->|VLAN 20| VLAN20
    PFSENSE -->|VLAN 30| VLAN30
    PFSENSE -->|VLAN 50| VLAN50
