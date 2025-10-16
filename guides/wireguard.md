# Introduction

## What is it and how it work

WireGuard is a VPN tunnel that use UDP, designed to be fast, secure and lightweight. It was created to replace other open source VPN technology like IPSec and OpenVPN and is in fact better in term of performance. We will see that it's very similar to SSH and almost as easy to deploy.

To establish a connection, two peers exchange their public key firstly to authenticate each other. If the public key does not match the one of the peer configured, no packet will be send back. After the authentication, each peer derive a shared secret with their private key and the other peer public key that will be used to encrypt packets. This shared secret is identical in both peer due to [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) key exchange property. The shared secret is then used to generate ephemeral session keys for encrypting and authenticating data.

Some popular commercial solution that use WireGuard are [Mullvad](https://mullvad.net/) and [Tailscale](https://tailscale.com)
## Some important security features

**A silent protocol :** When a WireGuard interface is exposed to the internet, it will not respond to anyone until a valid public key if presented. It means it's invisible to illegitimate peers and network scanners. Of course, that doesn't mean that it can't be breached but more like it will not be the main service targeted because of it's silent behavior.

![megamind meme](memes/megamind.jpg)

**Forward Secrecy :** WireGuard supports periodic re-keying, where new session keys are generated at regular intervals. This limits the amount of data that can be decrypted if a key is compromised.

**Post quantum consideration :** While WireGuard does not support post quantum cryptography, an extra layer of security can still be added by using a 256-bit Preshared Key. This way even if [Curve25519](https://en.wikipedia.org/wiki/Curve25519) used for key exchange is breached in the future, the Preshared Key used will also need to be compromised to decrypt the traffic.
## Use case and architecture

### Remote access

This setup will suit you in the case where you have a private resource that you want to access securely over an untrusted link. An example could be a company with employees in remote or just someone like me that want to access their Homelab securely over the internet.

In this architecture, only the server where WireGuard is installed will listen for incoming connection of one or more client. Of course, you will need to have your listening interface exposed to internet or at least accessible from your client computer depending of your setup.

![vpn-remote-client](../diagrams/svg/vpn-remote-client.svg)

### Site-to-site

The second setup is a classic site-to-site configuration where the goal is to make two private network communicate with each other. An example of this could be a company that want to make a remote site communicate with their main site so their employees can access private resources.

![vpn-remote-site](../diagrams/svg/vpn-remote-site.svg)

Another example of setup could be on a very high speed link between two data centre that you want to secure. Synacktiv, a French cybersecurity company made a great article on this : [Defend against vampires with 10 gbps network encryption](https://www.synacktiv.com/publications/defend-against-vampires-with-10-gbps-network-encryption)
# Deployment
## Installation

On Debian or Ubuntu :
```sh
sudo apt install wireguard
```

On Arch Linux :
```sh
sudo pacman -S wireguard-tools
```

It can also be installed on various other platform including Windows and Mac, see [Installation](https://www.wireguard.com/install/)
## Keys Generation

A private and public key need to be generated on each peer where you wish to use setup WireGuard. Be sure to keep the private key safe on the peer where it will be used, only the public key will be shared. A Preshared key is optional but I recommend it because it doesn't add any complexity to the configuration.

Generate a public and private key and write them in two different files :
```sh
wg genkey | tee privatekey | wg pubkey > publickey
```

Same for the Preshared Key (PSK) :
```sh
wg genpsk | tee psk
```

Now you should have three files :
1. privatekey
2. publickey
3. psk

## Configuration

WireGuard use what is called "cryptokey routing", It's an association between peers (identified by public key) and source IP addresses they are authorize to use. Now, we will review the variable in the configuration file. First, we have the Interface section to configure the local peer :

*PrivateKey* - The private key of the local peer
*Address* - The IP address of our local peer
*ListenPort* - The port where our local peer will listen for incoming connection
*PostUp* -  Command to execute when the interface goes up
*PostDown* - Command to execute when the interface goes down
*DNS* - Indicate a DNS server and domain suffix to use

Then we have the Peer section that can be duplicate for as many peer as we want:

*PublicKey* - The public key used to authenticate the remote peer
*PresharedKey* - The Preshared key
*AllowedIPs* - The address to which a node will route traffic
*Endpoint* - The address of the peer you want to connect to
*PersistentKeepalive* - To keep alive NAT-ed connection

For the following examples, we will consider that we are in a client-server architecture and that the [172.16.1.1.:51820] is the exposed address of our server that the client (or site) will use to access the server. Additionally, the internal DNS server of our fictional private network will be [10.10.50.2] and our domain suffix [capybara.local]. Note that the DNS configuration is completely optional. In the diagram, wg0 is the virtual interface created by WireGuard to manage traffic. WireGuard also support IPv6 but we will not present it here.
### Remote access

In this first setup, I will cover the configuration for the following remote access architecture :

![remote-access-example](../diagrams/svg/remote-access-example.svg)

#### Remote-client

The configuration file for the remote-client will look like this :

```sh
[Interface]
PrivateKey = <remote-client_privatekey>
Address = 10.20.1.10/32
DNS = 10.10.50.2, capybara.local

[Peer]
PublicKey = <server-publickey>
PresharedKey = <psk>
AllowedIPs = 10.10.0.0/16
Endpoint = 172.16.1.1:51820
PersistentKeepalive = 25
```

We have the DNS parameter set so our client will use the DNS and the suffix specified. In this configuration, we want our remote-client to access only the local resources of our private network. This is why the AllowedIPs parameter is set to the subnet of our private network. This is called split-tunneling, the internet traffic will still be sent trough the local gateway of our client. We also consider that our remote client is using source NAT to access the server and so, this is why we set the PersistentKeepalive parameter to 25 to keep our connection alive.

#### Server

Here is the configuration file for our server :

```sh
[Interface]
PrivateKey = <server_privatekey>
Address = 10.20.1.1/32
ListenPort = 51820
PostUp = <command>
PostDown = <command>

[Peer]
# Name = remote-client
PublicKey = <remote-client_publickey>
PresharedKey = <psk>
AllowedIPs = 10.10.1.10/32
```

On the server, the ListenPort is set so client will be able to connect. We also have the PostUp and PostDown that we can use to execute routing command but we will not discuss it here. On the peer section, we have the public key of our remote-client and his IP address set, this way, it will be able to access our server.

### Site-to-site

We will now cover a simple site-to-site setup. The server will act as the hub for one remote-site even tho we could add as many as our server support.

![site-to-site-example](../diagrams/svg/site-to-site-example.svg)

#### Remote-site

The configuration file for the remote-site will look like this :

```sh
[Interface]
PrivateKey = <remote-site_privatekey>
Address = 10.20.1.2/32

[Peer]
# Name = remote-site
PublicKey = <server-publickey>
PresharedKey = <psk>
AllowedIPs = 10.10.0.0/16
Endpoint = 172.16.1.1:51820
```

Here we don't have the DNS parameter because our remote-site will act as a gateway to access the resource of the private network.
#### Server

Here is the configuration file for our server :

```sh
[Interface]
PrivateKey = <server_privatekey>
Address = 10.20.1.1/32
ListenPort = 51820
PostUp = <command>
PostDown = <command>

[Peer]
PublicKey = <remote-client_publickey>
PresharedKey = <psk>
AllowedIPs = 10.11.0.10/16
```

We can see in this configuration that the peer "AllowedIPs" parameter is set to the subnet of the remote-site, this way all clients in the remote site will be able to access the private network.

The best command to use is wg-quick, it let you start / stop the WG service for the configuration file.

To start the service on based on the configuration file :
```sh
wg-quick start <int>
```

To start the service at startup :
```sh
systemctl enable wg-quick@<int>
```

You can use tcpdump on the interface for troubleshoot :
```sh
tcpdump -i <int>
```

## Sources

- [Official website](https://www.wireguard.com/)
- [Wikipedia page](https://en.wikipedia.org/wiki/WireGuard)
- [Whitepaper](https://www.wireguard.com/papers/wireguard.pdf)
- [IVPN - Implementing Quantum Resistance in WireGuard using PresharedKey](https://www.ivpn.net/knowledgebase/general/quantum-resistant-vpn-connections/)
