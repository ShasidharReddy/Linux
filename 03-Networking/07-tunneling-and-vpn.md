# Tunneling and VPN

> **📌 Disclaimer**: Any third-party logos, screenshots, or diagrams referenced in this document are used for educational purposes only. All trademarks belong to their respective owners.


This file groups SSH tunneling, overlay tunnel concepts, VPN implementation details, and scenario-driven examples from the original networking guide.

---

## 7.13 SSH port forwarding overview
Types:

- Local forwarding `-L`
- Remote forwarding `-R`
- Dynamic forwarding `-D`

## 7.14 Local port forwarding
Syntax:

```bash
ssh -L 8080:127.0.0.1:80 user@server
```

Meaning:

- Local port `8080`
- Forwards to remote `127.0.0.1:80`

Use cases:

- Access internal web UI
- Reach database consoles securely
- Temporary admin access without exposing ports

## 7.15 Remote port forwarding
Syntax:

```bash
ssh -R 8443:127.0.0.1:443 user@server
```

Use case:

- Publish a local service to the remote side

## 7.16 Dynamic forwarding with SOCKS proxy
```bash
ssh -D 1080 user@bastion
```

This creates a SOCKS proxy for tunneling application traffic.

---

## 7.24 TUN vs TAP
| Type | Layer | Use Case |
|---|---|---|
| TUN | Layer 3 | Routed VPNs |
| TAP | Layer 2 | Bridged VPNs, Ethernet emulation |

## 7.25 GRE and VXLAN overview
Advanced overlays include:

- GRE
- IPIP
- VXLAN
- GENEVE

Used in:

- Data center overlays
- Cloud networking
- SDN environments

## 7.26 VXLAN conceptual note
VXLAN extends Layer 2 over Layer 3 using VNI identifiers and UDP encapsulation.

Often used with:

- Hypervisors
- Kubernetes CNIs
- EVPN fabrics

---

## VPN

VPNs securely connect remote users, sites, or workloads over untrusted networks.

### 📸 VPN Tunnel Concept
```mermaid
graph LR
    A[Client] -->|Encrypted Tunnel| B[VPN Server]
    B --> C[Internet / Private Network]
    style A fill:#e1f5fe
    style B fill:#fff3e0
    style C fill:#e8f5e9
```
> *Source: Wikimedia Commons — VPN tunnel overview*

## 7.1 Common VPN use cases
- Remote user access
- Site-to-site connectivity
- Cloud-to-datacenter links
- Admin access to private networks
- Secure overlay networks

## 7.2 OpenVPN overview
OpenVPN is mature, flexible, and widely deployed.

Characteristics:

- TLS-based
- TUN or TAP modes
- User-space implementation
- Good compatibility

## 7.3 WireGuard overview
WireGuard is modern, lean, and high performance.

Characteristics:

- Simple configuration
- Strong cryptography
- Kernel integration on many platforms
- Excellent site-to-site and remote access fit

## 7.4 OpenVPN basic concepts
Files often used:

- CA certificate
- Server certificate
- Server key
- Client certificate
- Client key
- `ta.key` or TLS auth material in some setups

## 7.5 Example OpenVPN server config
```conf
port 1194
proto udp
dev tun
server 10.8.0.0 255.255.255.0
push "redirect-gateway def1"
push "dhcp-option DNS 1.1.1.1"
keepalive 10 120
persist-key
persist-tun
user nobody
group nogroup
cipher AES-256-GCM
verb 3
```

## 7.6 Example OpenVPN client config
```conf
client
dev tun
proto udp
remote vpn.example.com 1194
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
verb 3
```

## 7.7 Start OpenVPN service
Depending on distro packaging:

```bash
sudo systemctl enable --now openvpn-server@server
```

or:

```bash
sudo systemctl enable --now openvpn@server
```

## 7.8 OpenVPN firewall and forwarding
Typical requirements:

- Allow UDP 1194
- Enable IP forwarding
- NAT VPN client subnet to outbound interface if needed

Example:

```bash
sudo firewall-cmd --permanent --add-port=1194/udp
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --reload
```

## 7.9 WireGuard basic concepts
WireGuard uses:

- Interface-based configuration
- Public/private key pairs
- Allowed IPs for routing
- UDP transport

## 7.10 Generate WireGuard keys
```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

Protect the private key.

## 7.11 Example WireGuard server config
File:

```text
/etc/wireguard/wg0.conf
```

Example:

```ini
[Interface]
Address = 10.20.30.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
PostUp = iptables -t nat -A POSTROUTING -s 10.20.30.0/24 -o eth0 -j MASQUERADE
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp = iptables -A FORWARD -o wg0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -s 10.20.30.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -o wg0 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

[Peer]
PublicKey = CLIENT_PUBLIC_KEY
AllowedIPs = 10.20.30.2/32
```

## 7.12 Example WireGuard client config
```ini
[Interface]
Address = 10.20.30.2/24
PrivateKey = CLIENT_PRIVATE_KEY
DNS = 1.1.1.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = vpn.example.com:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

## 7.13 Start WireGuard
```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show
```

## 7.14 Split tunnel vs full tunnel
| Mode | Description |
|---|---|
| Split tunnel | Only selected networks go through VPN |
| Full tunnel | All traffic goes through VPN |

In WireGuard, `AllowedIPs` often controls routing behavior.

Examples:

- Split tunnel: `10.10.0.0/16, 10.20.0.0/16`
- Full tunnel: `0.0.0.0/0, ::/0`

## 7.15 Site-to-site VPN design
Typical considerations:

- Avoid overlapping subnets
- Define clear allowed networks
- Ensure return routes exist
- Permit forwarding and firewall rules
- Consider MTU and MSS clamping if needed

## 7.16 Verify VPN operation
Commands:

```bash
ip addr show wg0
ip route
wg show
ping -c 3 10.20.30.1
```

For OpenVPN:

```bash
ip addr show tun0
ss -lunp | grep 1194
```

## 7.17 Common VPN issues
- UDP port blocked
- Missing IP forwarding
- Missing NAT
- Wrong AllowedIPs
- MTU problems
- Clock skew affecting TLS-based VPNs
- DNS not pushed or configured correctly

## 7.18 Security best practices for VPNs
- Use modern crypto defaults
- Rotate keys or certificates
- Limit peer access
- Log connections responsibly
- Restrict management access to VPN endpoints
- Patch VPN software promptly

## 7.19 MTU and MSS tuning for VPNs
Encapsulation reduces effective MTU.

Symptoms:

- HTTPS stalls
- Intermittent app failure
- Large packets retransmit

Potential mitigation:

- Lower tunnel MTU
- Clamp TCP MSS at firewall or router

<a id="tunneling-vpn"></a>
## 🔒 Tunneling & VPN

VPNs are one expression of a broader tunneling concept: taking traffic that logically belongs on one network and carrying it across another path through encapsulation, encryption, or both. In Linux environments, tunnels are used for remote administration, site-to-site connectivity, overlay networking, private service access, and traffic engineering.

### What is Tunneling?

Tunneling means wrapping one protocol inside another so the original traffic can traverse an intermediate network.

Examples:

- SSH forwards a TCP service through an encrypted SSH session.
- GRE carries packets inside IP for routed overlays.
- IPsec encrypts IP traffic between gateways or hosts.
- WireGuard creates a lightweight encrypted UDP tunnel.
- OpenVPN carries IP packets over UDP or TCP in user space.
- VXLAN carries Layer 2 frames over Layer 3 UDP transport.

Why teams use tunnels:

- Encrypt traffic across untrusted networks
- Access private resources from remote locations
- Connect branch offices or data centers
- Extend a tenant network across hosts
- Carry management traffic separately from application traffic

### Common tunneling protocols at a glance

| Protocol | Encrypts Traffic | Layer | Typical Use |
|---|---|---|---|
| SSH tunnel | Yes | Application over TCP | Admin access, ad hoc secure forwarding |
| GRE | No | Layer 3 wrapper | Routed overlays, multicast carriage |
| IPsec | Yes | Layer 3 | Site-to-site or host-to-host VPN |
| WireGuard | Yes | Layer 3 over UDP | Modern VPN, simple peer meshes |
| OpenVPN | Yes | Layer 3 or Layer 2 | Remote access, cross-platform VPN |
| VXLAN | No by itself | Layer 2 over UDP | Data center overlays, container fabrics |

### SSH tunnel types

```mermaid
graph LR
    U["💻 Admin Laptop"] --> J["🪜 Jump Host"]
    J --> DB["🗄️ DB Server"]
    U --> L["🔁 Local Forward<br/>ssh -L 8080:db-server:3306 jump-host"]
    R["🌍 Remote User"] --> RH["🖥️ Remote Server"]
    RH --> RF["🔁 Remote Forward<br/>ssh -R 9090:localhost:3000 remote-server"]
    U --> D["🧦 Dynamic SOCKS<br/>ssh -D 1080 jump-host"]
```

### SSH Tunneling (Detailed)

SSH tunneling is often the fastest safe way to reach an internal service without exposing that service directly to the Internet.

#### Local port forwarding

Local forwarding binds a port on your machine and sends traffic through SSH to a destination reachable from the remote SSH server.

Command:

```bash
ssh -L 8080:db-server.corp.example.internal:3306 ops@jump1.corp.example.internal
```

What it means:

- Local laptop listens on `127.0.0.1:8080`
- SSH session terminates on `jump1.corp.example.internal`
- From the jump host, traffic is forwarded to `db-server.corp.example.internal:3306`

Use case:

- Access an internal MySQL database from your laptop without opening port 3306 publicly

Full example workflow:

```bash
ssh -N -L 3307:db1.corp.example.internal:3306 ops@bastion.example.com
mysql -h 127.0.0.1 -P 3307 -u app_readonly -p
```

Helpful flags:

- `-N` does not run a remote shell
- `-f` backgrounds the tunnel after authentication
- `-o ExitOnForwardFailure=yes` makes the command fail fast if the port cannot be bound

Safer production variant:

```bash
ssh -fN \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -L 3307:db1.corp.example.internal:3306 \
  ops@bastion.example.com
```

Verify the local listener:

```bash
ss -tlnp | grep 3307
nc -zv 127.0.0.1 3307
```

#### Remote port forwarding

Remote forwarding binds a port on the remote SSH server and sends traffic back to your local machine.

Command:

```bash
ssh -R 9090:localhost:3000 dev@remote-server.example.com
```

Use case:

- Expose a local development web server to a remote host during testing

Important considerations:

- The remote server may restrict `GatewayPorts`.
- `localhost:3000` refers to your local machine from the SSH client perspective.
- This can unintentionally expose a private service if the remote side binds widely.

More explicit remote bind example:

```bash
ssh -R 127.0.0.1:9090:localhost:3000 dev@remote-server.example.com
```

Then, on the remote server:

```bash
curl http://127.0.0.1:9090/
```

#### Dynamic port forwarding

Dynamic forwarding creates a SOCKS proxy.

Command:

```bash
ssh -D 1080 -N ops@jump1.corp.example.internal
```

Use case:

- Route browser or CLI traffic through the jump host without defining every destination explicitly

Example with `curl` through the SOCKS proxy:

```bash
curl --socks5-hostname 127.0.0.1:1080 https://ifconfig.me
```

Browser configuration example:

- Proxy type: SOCKS5
- Host: `127.0.0.1`
- Port: `1080`
- Remote DNS via SOCKS: enabled if the browser supports it

#### ProxyJump and multi-hop SSH

Modern OpenSSH makes multi-hop access much cleaner.

Direct command example:

```bash
ssh -J bastion1.example.com,bastion2.example.com app1.corp.example.internal
```

Equivalent SSH client config:

```sshconfig
Host bastion1
    HostName bastion1.example.com
    User ops
    IdentityFile ~/.ssh/ops_ed25519

Host bastion2
    HostName bastion2.example.com
    User ops
    IdentityFile ~/.ssh/ops_ed25519
    ProxyJump bastion1

Host app1
    HostName app1.corp.example.internal
    User ops
    IdentityFile ~/.ssh/ops_ed25519
    ProxyJump bastion2
```

With this config, the connection becomes simple:

```bash
ssh app1
scp ./bundle.tar.gz app1:/srv/releases/
```

#### Persistent tunnels with autossh

For long-lived tunnels, `autossh` can recreate the SSH session if it fails.

Example command:

```bash
autossh -M 0 -fN \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -L 8443:internal-api.corp.example.internal:443 \
  ops@bastion.example.com
```

Systemd service example for a persistent local forward:

```ini
[Unit]
Description=Persistent SSH tunnel to internal API
After=network-online.target
Wants=network-online.target

[Service]
User=ops
ExecStart=/usr/bin/autossh -M 0 -N \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -o ExitOnForwardFailure=yes \
  -L 8443:internal-api.corp.example.internal:443 \
  ops@bastion.example.com
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### GRE Tunnels

GRE, or Generic Routing Encapsulation, carries one Layer 3 payload inside another IP packet. GRE is lightweight and flexible but does not encrypt traffic by itself.

Common reasons to use GRE:

- Route private networks over a simple point-to-point overlay
- Carry multicast or nontrivial routing cases where pure IPsec is awkward
- Build lab overlays between Linux systems

Example tunnel endpoints:

- Server A public IP: `198.51.100.10`
- Server B public IP: `203.0.113.10`
- GRE inside addresses: `10.10.10.1/30` and `10.10.10.2/30`

On Server A:

```bash
sudo ip tunnel add gre1 mode gre remote 203.0.113.10 local 198.51.100.10 ttl 255
sudo ip addr add 10.10.10.1/30 dev gre1
sudo ip link set gre1 up
sudo ip route add 10.60.0.0/16 via 10.10.10.2 dev gre1
```

On Server B:

```bash
sudo ip tunnel add gre1 mode gre remote 198.51.100.10 local 203.0.113.10 ttl 255
sudo ip addr add 10.10.10.2/30 dev gre1
sudo ip link set gre1 up
sudo ip route add 10.50.0.0/16 via 10.10.10.1 dev gre1
```

Verification commands:

```bash
ip addr show dev gre1
ip route show
ping -c 3 10.10.10.2
traceroute 10.60.1.10
```

Operational notes:

- Open protocol 47 on intermediate firewalls because GRE is not TCP or UDP.
- Expect no confidentiality or integrity protection without a second layer such as IPsec.
- Account for MTU reduction due to encapsulation.

#### GRE over IPsec

A common design is GRE for routing flexibility plus IPsec for encryption.

Benefits:

- GRE can carry routing protocols or multicast
- IPsec protects the outer path
- The tunnel acts like a routed point-to-point link

### IPsec VPN

IPsec is a standards-based suite for securing IP traffic.

### 📸 IPsec Architecture
![IPsec](https://upload.wikimedia.org/wikipedia/commons/6/6b/Ipsec-ah.svg)
> *Source: Wikimedia Commons — IPsec Authentication Header*

Core terms:

- IKE: Internet Key Exchange, used to negotiate security associations
- ESP: Encapsulating Security Payload, used for encrypted payload transport
- AH: Authentication Header, less common today

#### Transport vs tunnel mode

| Mode | Protects | Common Use |
|---|---|---|
| Transport mode | Payload of the original packet | Host-to-host security |
| Tunnel mode | Entire original IP packet inside a new outer packet | Site-to-site VPN |

#### Site-to-site IPsec architecture

```mermaid
graph TD
    SITEA["🏢 Office LAN<br/>10.50.0.0/16"] --> GWA["🔐 IPsec Gateway A"]
    GWA --> INET["🌐 Internet"]
    INET --> GWB["🔐 IPsec Gateway B"]
    GWB --> SITEB["☁️ AWS VPC<br/>10.60.0.0/16"]
```

#### strongSwan site-to-site example

Example design:

- Office gateway public IP: `198.51.100.20`
- AWS gateway public IP: `203.0.113.20`
- Office LAN: `10.50.0.0/16`
- AWS VPC: `10.60.0.0/16`

Install strongSwan:

```bash
sudo apt update
sudo apt install -y strongswan
```

`/etc/ipsec.conf` on the office gateway:

```conf
config setup
    charondebug="ike 2, knl 2, cfg 2"

conn aws-site
    auto=start
    keyexchange=ikev2
    type=tunnel
    authby=psk
    left=198.51.100.20
    leftid=198.51.100.20
    leftsubnet=10.50.0.0/16
    leftfirewall=yes
    right=203.0.113.20
    rightid=203.0.113.20
    rightsubnet=10.60.0.0/16
    ike=aes256-sha256-modp2048!
    esp=aes256-sha256!
    dpdaction=restart
    dpddelay=30s
    ikelifetime=8h
    lifetime=1h
```

`/etc/ipsec.secrets`:

```conf
198.51.100.20 203.0.113.20 : PSK "replace-with-a-long-random-pre-shared-key"
```

Equivalent right-side configuration swaps `left` and `right` parameters.

Forwarding and firewall prerequisites:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo nft add rule inet filter input udp dport { 500, 4500 } accept
sudo nft add rule inet filter input ip protocol esp accept
sudo nft add rule inet filter forward ip saddr 10.50.0.0/16 ip daddr 10.60.0.0/16 accept
sudo nft add rule inet filter forward ip saddr 10.60.0.0/16 ip daddr 10.50.0.0/16 ct state established,related accept
```

Verification commands:

```bash
sudo ipsec restart
sudo ipsec statusall
sudo ip xfrm state
sudo ip xfrm policy
ping -c 3 10.60.1.10
```

Useful logs:

```bash
sudo journalctl -u strongswan -f
sudo journalctl -u charon -f
```

### WireGuard VPN (Complete Setup)

WireGuard is a modern UDP-based VPN that uses concise configuration files and modern cryptography.

Why teams like WireGuard:

- Small configuration surface
- Fast in kernel space on Linux
- Clear peer model
- Easy split tunnel or full tunnel design
- Works well for admin overlays and site links

Example design:

- Server public IP: `198.51.100.30`
- Tunnel network: `10.200.0.0/24`
- Server tunnel IP: `10.200.0.1/24`
- Linux laptop peer: `10.200.0.10/32`
- macOS laptop peer: `10.200.0.11/32`

Install WireGuard on the server:

```bash
sudo apt update
sudo apt install -y wireguard
```

Generate server keys:

```bash
umask 077
wg genkey | sudo tee /etc/wireguard/server.key | wg pubkey | sudo tee /etc/wireguard/server.pub
```

Generate a client key pair on Linux or macOS:

```bash
umask 077
wg genkey | tee client.key | wg pubkey > client.pub
```

`/etc/wireguard/wg0.conf` on the server:

```ini
[Interface]
Address = 10.200.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
SaveConfig = false

[Peer]
PublicKey = LINUX_CLIENT_PUBLIC_KEY
AllowedIPs = 10.200.0.10/32,10.50.10.0/24
PersistentKeepalive = 25

[Peer]
PublicKey = MAC_CLIENT_PUBLIC_KEY
AllowedIPs = 10.200.0.11/32
PersistentKeepalive = 25
```

Recommended persistent sysctl file:

```bash
cat <<'EOF' | sudo tee /etc/sysctl.d/99-wireguard-forwarding.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

Example firewall rules for a WireGuard gateway:

```nft
table inet filter {
  chain input {
    type filter hook input priority 0;
    policy drop;
    iifname "lo" accept
    ct state established,related accept
    udp dport 51820 accept
    tcp dport 22 accept
  }

  chain forward {
    type filter hook forward priority 0;
    policy drop;
    ct state established,related accept
    iifname "wg0" accept
    oifname "wg0" accept
  }
}

table ip nat {
  chain postrouting {
    type nat hook postrouting priority 100;
    ip saddr 10.200.0.0/24 oifname "ens160" masquerade
  }
}
```

Safer server config without dynamic firewall edits:

```ini
[Interface]
Address = 10.200.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
SaveConfig = false

[Peer]
PublicKey = LINUX_CLIENT_PUBLIC_KEY
AllowedIPs = 10.200.0.10/32
PersistentKeepalive = 25

[Peer]
PublicKey = MAC_CLIENT_PUBLIC_KEY
AllowedIPs = 10.200.0.11/32
PersistentKeepalive = 25
```

Start and enable the interface:

```bash
sudo systemctl enable --now wg-quick@wg0
sudo wg show
ip addr show wg0
```

Linux client config example:

```ini
[Interface]
Address = 10.200.0.10/32
PrivateKey = CLIENT_PRIVATE_KEY
DNS = 10.20.30.53

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = vpn1.example.com:51820
AllowedIPs = 10.20.30.0/24,10.50.0.0/16,10.200.0.1/32
PersistentKeepalive = 25
```

Full-tunnel Linux client example:

```ini
[Interface]
Address = 10.200.0.10/32
PrivateKey = CLIENT_PRIVATE_KEY
DNS = 10.20.30.53

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = vpn1.example.com:51820
AllowedIPs = 0.0.0.0/0,::/0
PersistentKeepalive = 25
```

macOS client notes:

- Install the official WireGuard client.
- Import the same configuration with the client private key and the server public key.
- Ensure `AllowedIPs` matches the intended split or full tunnel behavior.
- If DNS should use internal resolvers, specify them in the client profile.

#### Peer design guidance

`AllowedIPs` means two things in WireGuard:

- Which traffic should be routed into the tunnel
- Which source prefixes are accepted from that peer

That dual meaning makes correctness critical.

Examples:

- Remote admin laptop: `AllowedIPs = 10.200.0.10/32`
- Branch office behind a peer: `AllowedIPs = 10.200.0.20/32,10.70.0.0/16`
- Full-tunnel client: `AllowedIPs = 0.0.0.0/0,::/0`

#### WireGuard mesh topology

```mermaid
graph TD
    HUB["🔐 WG Hub<br/>10.200.0.1"] --> A["💻 Admin Laptop<br/>10.200.0.10"]
    HUB --> B["💻 MacBook<br/>10.200.0.11"]
    HUB --> C["🏢 Branch Gateway<br/>10.200.0.20"]
    C --> LAN["🌐 Branch LAN<br/>10.70.0.0/16"]
```

#### WireGuard troubleshooting

State inspection:

```bash
sudo wg show
ip route
ip rule
```

Check the UDP listener:

```bash
ss -lunp | grep 51820
```

Observe encrypted traffic on the underlay:

```bash
sudo tcpdump -ni ens160 udp port 51820
```

Observe cleartext traffic inside the tunnel:

```bash
sudo tcpdump -ni wg0
```

Common issues:

- Public UDP port blocked
- Wrong server endpoint name or IP
- Incorrect peer `AllowedIPs`
- Missing return route behind a site peer
- MTU too high on unstable paths
- Clock issues are rare for WireGuard itself but matter for surrounding DNS or auth tooling

### OpenVPN Setup

OpenVPN remains useful when you need broad client compatibility, mature certificate workflows, or TCP mode in constrained environments.

Install packages:

```bash
sudo apt update
sudo apt install -y openvpn easy-rsa
```

Example server config `/etc/openvpn/server/server.conf`:

```conf
port 1194
proto udp
dev tun
user nobody
group nogroup
persist-key
persist-tun
server 10.8.0.0 255.255.255.0
topology subnet
ifconfig-pool-persist /var/log/openvpn/ipp.txt
push "route 10.20.30.0 255.255.255.0"
push "route 10.50.0.0 255.255.0.0"
push "dhcp-option DNS 10.20.30.53"
keepalive 10 60
cipher AES-256-GCM
data-ciphers AES-256-GCM:AES-128-GCM
auth SHA256
tls-server
ca /etc/openvpn/pki/ca.crt
cert /etc/openvpn/pki/issued/server.crt
key /etc/openvpn/pki/private/server.key
dh none
ecdh-curve prime256v1
tls-crypt /etc/openvpn/pki/ta.key
status /var/log/openvpn/status.log
verb 3
explicit-exit-notify 1
```

Certificate generation with Easy-RSA:

```bash
make-cadir ~/easy-rsa
cd ~/easy-rsa
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa build-client-full laptop01 nopass
openvpn --genkey secret ta.key
```

Copy outputs into the server paths expected by `server.conf`.

Client config example `laptop01.ovpn`:

```conf
client
dev tun
proto udp
remote vpn1.example.com 1194
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3
<ca>
-----BEGIN CERTIFICATE-----
REPLACE_WITH_CA_CERT
-----END CERTIFICATE-----
</ca>
<cert>
-----BEGIN CERTIFICATE-----
REPLACE_WITH_CLIENT_CERT
-----END CERTIFICATE-----
</cert>
<key>
-----BEGIN PRIVATE KEY-----
REPLACE_WITH_CLIENT_KEY
-----END PRIVATE KEY-----
</key>
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
REPLACE_WITH_TLS_CRYPT_KEY
-----END OpenVPN Static key V1-----
</tls-crypt>
```

Start the server:

```bash
sudo systemctl enable --now openvpn-server@server
sudo systemctl status openvpn-server@server
```

If your distro uses a different unit name, verify with:

```bash
systemctl list-unit-files | grep -i openvpn
```

OpenVPN verification commands:

```bash
ip addr show tun0
ss -lunp | grep 1194
sudo journalctl -u openvpn-server@server -f
```

### VXLAN

VXLAN, or Virtual Extensible LAN, carries Layer 2 Ethernet segments across a Layer 3 network by encapsulating frames in UDP, commonly on port 4789.

Typical use cases:

- Data center overlays
- Hypervisor fabrics
- Kubernetes or container networking backends
- Extending tenant segments without stretching physical VLANs everywhere

Example point-to-point VXLAN on Linux:

Server A:

```bash
sudo ip link add vxlan10 type vxlan id 10 dev ens160 remote 203.0.113.50 local 198.51.100.50 dstport 4789
sudo ip addr add 10.250.10.1/24 dev vxlan10
sudo ip link set vxlan10 up
```

Server B:

```bash
sudo ip link add vxlan10 type vxlan id 10 dev ens160 remote 198.51.100.50 local 203.0.113.50 dstport 4789
sudo ip addr add 10.250.10.2/24 dev vxlan10
sudo ip link set vxlan10 up
```

Bridged VXLAN example for Layer 2 extension:

```bash
sudo ip link add br10 type bridge
sudo ip link set br10 up
sudo ip link set vxlan10 master br10
sudo ip link set eth1 master br10
```

Operational cautions:

- VXLAN does not encrypt by itself.
- MTU planning is critical because of additional encapsulation overhead.
- In real fabrics, dynamic endpoint discovery is often handled by EVPN rather than static `remote` definitions.

### Production troubleshooting checklist for tunnels and VPNs

1. Confirm the underlay path works first.
2. Verify the correct protocol and port are permitted.
3. Check whether the problem is encryption, routing, or DNS after connection.
4. Inspect interface state and routes on both ends.
5. Capture traffic on both the tunnel interface and the physical interface.
6. Validate MTU and MSS when large transfers fail but pings succeed.
7. Confirm return routes for site-to-site networks.
8. Review service logs before changing crypto settings blindly.

## 7.20 Summary
OpenVPN offers flexibility and broad compatibility. WireGuard offers simplicity and performance. Both require clean routing, forwarding, and firewall design.

---

# Tunneling and VPN Scenarios

## 7.2 Configure site-to-site VPN between office and AWS
Objective:

- Connect office subnet `10.50.0.0/16` to AWS subnet `10.60.0.0/16`
- Keep routes private without exposing workloads publicly

Implementation outline:

1. Select IPsec or WireGuard based on platform and policy requirements.
2. Ensure non-overlapping CIDRs.
3. Open the required ports and protocols on both edges.
4. Add routes on both sides.
5. Validate bidirectional reachability.

AWS-side considerations:

- VPC route tables must point `10.50.0.0/16` to the VPN gateway or tunnel appliance.
- Security groups and NACLs must allow the intended traffic.
- If an EC2 instance acts as the VPN router, disable source or destination checks.

Office-side validation:

```bash
ping -c 3 10.60.1.10
traceroute 10.60.1.10
ip route
sudo tcpdump -ni eth0 host 203.0.113.20
```

---

## 7.8 Provide administrators with a WireGuard management overlay
Objective:

- Reach SSH, monitoring, and web consoles over private tunnel addresses
- Avoid publishing every service directly

Design example:

- WireGuard hub: `10.200.0.1`
- Admin laptops: `10.200.0.10-20`
- Only management subnets are routed through the tunnel

Checks:

```bash
wg show
ping -c 3 10.200.0.1
ssh ops@10.20.30.21
curl -k https://10.20.30.51:3000/
```

---

## 7.11 Troubleshoot: Site-to-site tunnel comes up but subnet traffic still fails
Checklist:

```bash
ip route
sudo ip xfrm state
sudo ip xfrm policy
ping -c 3 <remote-gateway-inside-ip>
ping -c 3 <remote-host>
sudo tcpdump -ni any host <remote-host>
```

Most common causes:

- Interesting traffic selectors or `AllowedIPs` do not match the real subnet
- Local LAN host uses the wrong default gateway
- Return route missing on remote LAN
- Host firewall blocks forwarded traffic after decryption

---

## C.9 Exercise 9: Validate VPN routing logic

Questions:

- Which routes appear when the VPN comes up?
- Which DNS settings change?
- Is traffic split or full tunnel?

---

## X.3 Minimal VPN gateway checklist

1. Assign reachable public IP.
2. Open VPN UDP or TCP port.
3. Enable forwarding.
4. Add NAT if clients need Internet access.
5. Push or define DNS settings.
6. Verify client route table.
7. Verify tunnel IP connectivity.
