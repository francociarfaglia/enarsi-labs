# Site-to-Site VPN Deployment using Cisco VTI (IPsec)

This topology demonstrates the configuration of a secure Site-to-Site VPN tunnel utilizing Cisco’s **Virtual Tunnel Interface (VTI)**. Unlike traditional crypto maps, VTI simplifies configuration and supports dynamic routing protocols across the tunnel.

![alt text](<Images/VTI topology.png>)

## 1. Establishing Layer 3 Connectivity
The primary requirement is to ensure stable IP reachability between the peer routers before initiating the VPN.
* **R1 WAN Interface:** `10.0.0.2`
* **R3 WAN Interface:** `10.0.0.6`

![alt text](<Images/ip connectivity verification.png>)

---

## 2. Phase 1: IKE / ISAKMP Configuration
The first phase of the negotiation involves establishing a secure management channel by defining the **ISAKMP (Internet Security Association and Key Management Protocol)** policy.

```
crypto isakmp policy 10
 encr aes
 authentication pre-share
 group 14
```

![alt text](<Images/isakmp policy R1.png>)
![alt text](<Images/isakmp policy R3.png>)


> **Security Note:** All parameters (Encryption, Hash, Diffie-Hellman group) must match exactly on both peers. The **lifetime** value is the only exception, if mismatched, the routers will automatically negotiate to the lowest configured value.

### Authentication
For this lab, we use a **Pre-Shared Key (PSK)** to authenticate the peers.

On R1:

```
crypto isakmp key SECRETKEY address 10.0.0.6 
```

On R3:

```
crypto isakmp key SECRETKEY address 10.0.0.2 
```


![alt text](<Images/pre-sahred key R1.png>) 
![alt text](<Images/pre-sahred key R3.png>)

Upon successful configuration of the PSK and policy, Phase 1 is complete.

---

## 3. Phase 2: IPsec Transform-Set & Profile
Phase 2 defines how the actual data will be protected. We configure a **Transform-Set** to specify the encryption and integrity algorithms. For VTI, we utilize **Tunnel Mode**.

```
crypto ipsec transform-set VTISET ah-sha-hmac esp-aes 
 mode tunnel
```

![alt text](<Images/transform-set R1.png>)
![alt text](<Images/transform-set R3.png>)

### IPsec Profile Creation
To bridge the Transform-Set with the Tunnel Interface, we create an **IPsec Profile**. This profile acts as a template for the tunnel's security parameters.

```
crypto ipsec profile VTIPROFILE
 set transform-set VTISET 
```

![alt text](<Images/vti profile R1.png>)
![alt text](<Images/vti profile R3.png>)

With the profile defined, the Phase 2 parameters are fully established.

---

## 4. Virtual Tunnel Interface (VTI) Deployment
The final step in the VPN setup is the creation of the logical **Tunnel Interface**. By applying the IPsec profile directly to the tunnel, we eliminate the need for complex access lists (ACLs) used in legacy crypto-map configurations.

On R1:

```
interface Tunnel10
 ip address 172.16.0.1 255.255.255.252
 tunnel source 10.0.0.2
 tunnel mode ipsec ipv4
 tunnel destination 10.0.0.6
 tunnel protection ipsec profile VTIPROFILE
```

On R3

```
interface Tunnel10
 ip address 172.16.0.2 255.255.255.252
 tunnel source 10.0.0.6
 tunnel mode ipsec ipv4
 tunnel destination 10.0.0.2
 tunnel protection ipsec profile VTIPROFILE
```

![alt text](<Images/tunnel 10 interface R1.png>)
![alt text](<Images/tunnel 10 interface R3.png>)

### Verification of Security Associations
Once the interfaces are brought up, we can verify the state of the IPsec sessions.

![alt text](<Images/session R1.png>)
![alt text](<Images/session R3.png>)

---

## 5. Dynamic Routing over IPsec (OSPF)
With the VTI tunnel active, the routers treat the link as a point-to-point connection. This allows for the seamless exchange of routing information using **OSPF**. 

The routers form an adjacency over the tunnel, allowing internal networks (such as Loopback interfaces) to be shared securely.

![alt text](<Images/ospf R1.png>)
![alt text](<Images/ospf R3.png>)

**Observation:** As shown in the routing table, the next hop for remote networks is the **Tunnel interface** rather than the physical WAN interface.

### End-to-End Connectivity Test
Successful communication between Loopback interfaces confirms that traffic is being correctly encapsulated and encrypted.

![alt text](Images/ping.png)