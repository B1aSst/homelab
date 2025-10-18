# Homelab

<!--toc:start-->
- [Homelab](#homelab)
  - [Hardware](#hardware)
  - [Diagrams](#diagrams)
  - [Roadmap](#roadmap)
<!--toc:end-->

**Homelab**
: *A server set up in one's home for the purpose of testing various configurations of hardware, operating systems, etc.*

My homelab is a project that started with a simple NAS in 2023 during my first internship as a sysadmin.
During a converstation, I discovered that they all had some servers to test things and practice so I simply said why not try myself ?

My only goal with this HomeLab is to be able to try things however I want and to learn. As my main topic of interest is network security, I'm very happy I started this HomeLab.

I tried to buy hardware with a minimalist approach, I don't wanted things too noisy or too big. It had to fit in a bookshelves and be in a living room.

## Hardware

**List :**
- Fortigate 60E (Firewall)
- Cisco SG300-10 (Switch L2)
- Synology DS223 (NAS)
- Minisforum NAB5 (Mini server)
- Ubiquiti UniFi UAP-nanoHD (Wi-Fi Access Point)
- APC Back-UPS (Battery backup & Surge protector)

I will try to explain my choice as much as possible but some just don't have explanation...

It started with the Synology NAS, today I think it was a mistake but I had to start somewhere. I think it was too expensive for my usage and today I only use it purely as a NAS but it can be used as a home server as it support Docker and has a lot of app built-in.

I got the fortigate, switch, and AP from second-hand. No need to buy them full price as I'm not a company... The fortigate is a great appliance and nice to have because I use them at work and so I'ts nice to practice. I'ts mainly to filter traffic between VLANs. The AP is use to diffuse VLAN in the house. The NAB5 mini pc because I simply wanted something powerfull, small and not too noisy. It has a great CPU and 32GB of RAM, it run Proxmox VE with several VM/LXC.
There is power cut sometimes so the UPS makes my lab not go down.

## Diagrams

D2 guide [here](/diagrams/diagram.md).

Here is my physical setup :
![physical diagram](diagrams/svg/network.svg)

## Services

### Network

To-do...

## Guides

Some guides [here](/guides)

## Roadmap

- [ ] PVE Cluster
- [ ] Automate VM/LXC creation
- [ ] Multi sites topology
