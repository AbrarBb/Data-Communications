# Cisco Packet Tracer: Dynamic Routing Lab (RIPv2, EIGRP, OSPF)

Configuration guide for building a 4-router topology in Cisco Packet Tracer and testing **RIPv2**, **EIGRP**, and **OSPF** dynamic routing protocols.

---

## 1. Build the Topology

* **Devices:** `4 × Router-PT`, `4 × Switch-PT`, `4 × PC-PT`
* **Serial Modules:** The routers need serial ports. Power each router off, add a `PT-ROUTER-NM-1S` module to two slots so you get `Se2/0` and `Se3/0`, then power it back on.
* **Cables:**
  * **Copper Straight-Through** between PC, switch, and router (`Fa0` on the PC, `Fa0/1` on the switch, `Fa0/0` on the router, as drawn).
  * **Serial DCE (red)** between routers.
  * Make **Router0** and **Router3** the **DCE** side (the clock icons in your diagram).

---

## 2. Addressing Plan

### Network Subnets & Links

| Network | Subnet Mask | Connection |
| :--- | :--- | :--- |
| `192.10.11.0` | `255.255.255.0` | Router0 `Fa0/0` ↔ Switch0 ↔ PC0 |
| `62.0.0.0` | `255.0.0.0` | Router1 `Fa0/0` ↔ Switch1 ↔ PC1 |
| `162.30.0.0` | `255.255.0.0` | Router3 `Fa0/0` ↔ Switch3 ↔ PC2 |
| `163.33.0.0` | `255.255.0.0` | Router2 `Fa0/0` ↔ Switch2 ↔ PC3 |
| `125.0.0.0` | `255.0.0.0` | Router0 `Se2/0` ↔ Router1 `Se2/0` |
| `126.0.0.0` | `255.0.0.0` | Router0 `Se3/0` ↔ Router2 `Se3/0` |
| `127.0.0.0` | `255.0.0.0` | Router2 `Se2/0` ↔ Router3 `Se3/0` |
| `128.0.0.0` | `255.0.0.0` | Router3 `Se2/0` ↔ Router1 `Se3/0` |

### PC Configuration (`Desktop → IP Configuration`)

| PC | IP Address | Subnet Mask | Default Gateway |
| :--- | :--- | :--- | :--- |
| **PC0** | `192.10.11.2` | `255.255.255.0` | `192.10.11.1` |
| **PC1** | `62.0.0.2` | `255.0.0.0` | `62.0.0.1` |
| **PC2** | `162.30.0.2` | `255.255.0.0` | `162.30.0.1` |
| **PC3** | `163.33.0.2` | `255.255.0.0` | `163.33.0.1` |

---

## 3. Interface Configuration (Common to All Protocols)

### Router0
```ios
enable
conf t
hostname Router0
interface fa0/0
 ip address 192.10.11.1 255.255.255.0
 no shutdown
interface se2/0
 ip address 125.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
interface se3/0
 ip address 126.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
end
```

### Router1
```ios
enable
conf t
hostname Router1
interface fa0/0
 ip address 62.0.0.1 255.0.0.0
 no shutdown
interface se2/0
 ip address 125.0.0.2 255.0.0.0
 no shutdown
interface se3/0
 ip address 128.0.0.2 255.0.0.0
 no shutdown
end
```

### Router2
```ios
enable
conf t
hostname Router2
interface fa0/0
 ip address 163.33.0.1 255.255.0.0
 no shutdown
interface se2/0
 ip address 127.0.0.1 255.0.0.0
 no shutdown
interface se3/0
 ip address 126.0.0.2 255.0.0.0
 no shutdown
end
```

### Router3
```ios
enable
conf t
hostname Router3
interface fa0/0
 ip address 162.30.0.1 255.255.0.0
 no shutdown
interface se2/0
 ip address 128.0.0.1 255.0.0.0
 clock rate 64000
 no shutdown
interface se3/0
 ip address 127.0.0.2 255.0.0.0
 clock rate 64000
 no shutdown
end
```

---

## 4. Dynamic Routing

> **Note:** Task says “three routers”, but the diagram has four, so apply the protocol on all four. Configure **one protocol at a time**, test, then remove it before trying the next. Do not run them together.

### A) RIP (v2)

```ios
! Router0
conf t
router rip
 version 2
 no auto-summary
 network 192.10.11.0
 network 125.0.0.0
 network 126.0.0.0
end

! Router1
conf t
router rip
 version 2
 no auto-summary
 network 62.0.0.0
 network 125.0.0.0
 network 128.0.0.0
end

! Router2
conf t
router rip
 version 2
 no auto-summary
 network 163.33.0.0
 network 126.0.0.0
 network 127.0.0.0
end

! Router3
conf t
router rip
 version 2
 no auto-summary
 network 162.30.0.0
 network 127.0.0.0
 network 128.0.0.0
end
```
> **Cleanup:** To remove RIP before configuring the next protocol, run `no router rip` in global configuration mode (`conf t`) on every router.

---

### B) EIGRP (AS 100)

```ios
! Router0
conf t
router eigrp 100
 no auto-summary
 network 192.10.11.0 0.0.0.255
 network 125.0.0.0
 network 126.0.0.0
end

! Router1
conf t
router eigrp 100
 no auto-summary
 network 62.0.0.0
 network 125.0.0.0
 network 128.0.0.0
end

! Router2
conf t
router eigrp 100
 no auto-summary
 network 163.33.0.0
 network 126.0.0.0
 network 127.0.0.0
end

! Router3
conf t
router eigrp 100
 no auto-summary
 network 162.30.0.0
 network 127.0.0.0
 network 128.0.0.0
end
```
> **Cleanup:** To remove EIGRP before configuring the next protocol, run `no router eigrp 100` in global configuration mode (`conf t`) on every router.

---

### C) OSPF (Process 1, Area 0)

```ios
! Router0
conf t
router ospf 1
 network 192.10.11.0 0.0.0.255 area 0
 network 125.0.0.0 0.255.255.255 area 0
 network 126.0.0.0 0.255.255.255 area 0
end

! Router1
conf t
router ospf 1
 network 62.0.0.0 0.255.255.255 area 0
 network 125.0.0.0 0.255.255.255 area 0
 network 128.0.0.0 0.255.255.255 area 0
end

! Router2
conf t
router ospf 1
 network 163.33.0.0 0.0.255.255 area 0
 network 126.0.0.0 0.255.255.255 area 0
 network 127.0.0.0 0.255.255.255 area 0
end

! Router3
conf t
router ospf 1
 network 162.30.0.0 0.0.255.255 area 0
 network 127.0.0.0 0.255.255.255 area 0
 network 128.0.0.0 0.255.255.255 area 0
end
```
> **Cleanup:** To remove OSPF, run `no router ospf 1` in global configuration mode (`conf t`) on every router.

---

## 5. Verification

1. On each router, run `show ip route` (look for `R`, `D`, or `O` entries) and `show ip protocols`.
2. Wait about **30–60 seconds** after configuring for the routing protocol to converge.
3. Test end-to-end connectivity from the PC command prompts:

```bash
PC0> ping 62.0.0.2
PC0> ping 162.30.0.2
PC0> ping 163.33.0.2
PC1> ping 163.33.0.2
PC2> ping 192.10.11.2
```
