# Multiarea OSPF with Route Default Injection and Route Summarization

This report documents the implementation of a Multiarea OSPF topology, focusing on conditional default route injection from an ISP and inter-area route summarization at the ABR to optimize the routing table and prevent loops.

![alt text](Images/(1)Topology.png)

## 1. Topology and Initial Configuration

The topology consists of seven internal routers and an eighth representing the ISP border router. The internal network is divided into two areas: **Area 0** and **Area 1**.

* **Addressing:** Most internal networks utilize a `/29` mask, allowing each router to use its ID number as the last octet for its interfaces.
* **OSPF Configuration:** OSPF was configured on all inner-facing interfaces. The Router ID (RID) was manually configured using the router number as a guide (e.g., R1 has an OSPF RID of `1.1.1.1`).
* **Network Types:** Since all links are Ethernet, OSPF defaults to the **Broadcast** network type. To optimize adjacency speed, all links (except the `172.16.0.0/24` network shared by R5 and R7) were manually configured as **Point-to-Point**.

### Example: ABR (R1) OSPF Configuration

![alt text](<Images/(2)R1 OSPF configuration.png>)

To simulate networks for summarization, three loopbacks were configured on R2 and shared via OSPF:

![alt text](Images/(3)Loopbacks.png)

---

## 2. BGP Connectivity and Default Route Injection

An eBGP adjacency was created between **R6** and the **ISP**. A default route was advertised from the ISP to R6 using the `default-originate` command.

**R6 BGP Configuration:**

![alt text](<Images/(4)R6 BGP configuration.png>)

**ISP BGP Configuration:**

![alt text](<Images/(5)ISP BGP configuration.png>)

On R6, we can see the default route obtained via BGP in the routing table:

![alt text](<Images/(6)BGP default route present on R6.png>)
---

## 3. Conditional OSPF Default Route Injection

Instead of standard redistribution, this lab utilizes the OSPF **origination** mechanism to propagate a default route from R6 to internal routers. To ensure high availability and prevent traffic blackholing, a **Route-Map** is used to advertise the default route only when the physical link between R6 and the ISP is active.

1.  **Prefix-list:** Created to match the specific network between R6 and the ISP:

    ![alt text](Images/(7)Prefix-list.png)

2.  **Route-map:** Configured with a matching rule for the aforementioned prefix-list:

    ![alt text](Images/(8)Route-map.png)

3.  **OSPF Configuration:** The following command is applied in OSPF configuration mode:
    ```bash
    default-information originate always route-map [NAME]
    ```

    ![alt text](<Images/(9)Default information originate command.png>)

> **Logic:** The `default-information originate` command triggers the origination of a Type 5 LSA representing `0.0.0.0/0`. Adding the `always` keyword ensures this LSA is generated regardless of whether a default route exists in the RIB. Finally, the `route-map` adds a conditional check: the LSA is only advertised if the directly connected network between R6 and the ISP is present in the routing table.

### Verifying the Default Route
As long as the link to the internet is functional, R6 originates a **Type 5 (External) LSA**. Here is the R3 routing database showing the originated LSA:

![alt text](<Images/(10)Type 5 LSA on R3.png>)

In the routing table, we see an external **Type 2** OSPF default route (**O*E2**):

![alt text](<Images/(11)Default route on R3 routing table.png>)

---

## 4. External Metric Types (E1 vs E2)

It is interesting to notice that the cost of the default route is **1**, even though it has traversed several routers with higher-cost interfaces to reach R3. This is because OSPF originates external routes as **Metric Type 2** by default. For Type 2 routes, the cost is strictly defined by the seed metric (which is 1 by default) and does not increment as the LSA traverses the OSPF domain.

If the metric type is changed to **1**, the internal path cost is added to the seed metric. To test this, we can modify the origination parameters on R6:

![alt text](<Images/(12) Metric type 1.png>)

Now, the route appears as an external **Type 1** (**O*E1**) with a cumulative cost (e.g., 31), reflecting the total path metric from R3 to the ASBR (R6):

![alt text](<Images/(13)Type 1 default route on R3 routing table .png>)

## 5. End-to-End Connectivity Test

Once the topology has converged, a successful ping can be performed between all networks. Below is a ping between Loopback 0 on R2 and the Ethernet 0/0 interface on R6:

![alt text](<Images/(14)Successful ping.png>)
---

## 6. Inter-Area Route Summarization

If we look at the routing table of any Area 0 router, we see three OSPF **Inter-area (O IA)** entries representing the three `/24` loopbacks shared by R2. For example, on R5:

![alt text](<Images/(15)Non-summarized loopbacks.png>)

### Configuring the Summary
To summarize these into a single `/22` route, we must configure a summarization rule on **R1 (the ABR)**. It is important to keep in mind that summarized routes can only be inter-area or external, never intra-area. This is because all routers in a single area must have a complete, identical map of that area's topology due to the link-state nature of OSPF.

On R1, we enter:
```
R1(config-router)# area 1 range 192.168.0.0 255.255.252.0
```

### Summary Results

R5 now shows a single summarized **/22** route:

![alt text](<Images/(16)Summarized route.png>)
---

## 7. Loop Prevention via Null0

When the ABR creates the summarized route, a new entry is automatically added to its routing table that points to a non-existing interface: **Null0**.

![alt text](<Images/(17)Summarized route pointing to Null0.png>)

This mechanism exists for **loop prevention**. Because a summary route can cover more IP space than the ABR actually has as specific routes, a routing loop could otherwise form between the ABR and the core router.

When a packet arrives destined for an address like `192.168.3.1`—which falls within a summarized `/22` range but does not represent a real subnet in the local area—a routing loop can occur if a **Null0** interface is not utilized. Without a specific route to match the destination, the ABR (R1) reverts to its default route and forwards the traffic toward the core router. However, since the core router has the summarized `/22` route pointing back to R1 as the gateway for that entire range, it immediately returns the packet. This creates a continuous loop where the packet bounces between R1 and the core router until the TTL expires, unnecessarily consuming bandwidth and CPU resources.

### The Solution

With the summarized route installed and pointing to **Null0**, R1 matches the destination against the summary prefix and immediately discards the packet.

Because the Null0 route is more specific than the default route—and no valid longer-prefix match exists—the packet is dropped locally. This prevents routing loops and ensures efficient and predictable traffic handling.