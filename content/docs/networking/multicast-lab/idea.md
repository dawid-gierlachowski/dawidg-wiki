---
title: "Multicast Lab - Idea"
weight: 0
---

# The Concept

Simulate a company's internal network distributing 5 video channels to employees. A single source streams all 5 channels simultaneously — each host subscribes only to the 2 it needs. The network delivers traffic exclusively where it has been requested, nowhere else.

{{< hint info >}}
Built with **Containerlab**: FRR as the core router and Rendezvous Point, Arista cEOS as distribution switches, Linux containers as source and receivers.
{{< /hint >}}

---

## Multicast Groups

| Group | Channel |
|-------|---------|
| `239.1.1.1` | Channel A |
| `239.1.1.2` | Channel B |
| `239.1.1.3` | Channel C |
| `239.1.1.4` | Channel D |
| `239.1.1.5` | Channel E |

---

## Receiver Subscriptions

| Host | Joins |
|------|-------|
| HOST-1 | A, B |
| HOST-2 | B, C |
| HOST-3 | C, D |
| HOST-4 | D, E |
| HOST-5 | A, E |

Channel B reaches two different network branches. Channel A reaches two hosts on opposite sides of the topology. This overlap is intentional — it lets you verify the multicast tree is built correctly and traffic only flows where it should.

---

## Topology

```mermaid
graph TD
    SRC["SOURCE"]
    CORE["CORE-R\nFRR · Rendezvous Point"]
    D1["DIST-1\nArista cEOS"]
    D2["DIST-2\nArista cEOS"]
    H1["HOST-1\nA, B"]
    H2["HOST-2\nB, C"]
    H3["HOST-3\nC, D"]
    H4["HOST-4\nD, E"]
    H5["HOST-5\nA, E"]

    SRC --> CORE
    CORE --> D1
    CORE --> D2
    D1 --> H1
    D1 --> H2
    D2 --> H3
    D2 --> H4
    D2 --> H5
```

---

## Streaming Approach

Python scripts — simpler in containers, more instructive than real video.

{{< columns >}}
**Sender**

5 threads, each blasting UDP multicast to its group:

```
FEED-A | seq=1042 | ts=1709584823.4
```

<--->
**Receiver**

Binds to its 2 groups and prints incoming packets. Only sees what it joined.
{{< /columns >}}

{{< hint info >}}
Swap in `ffmpeg` with test-pattern video later if a visual demo is wanted.
{{< /hint >}}

---

## What the Lab Proves

{{< details "Verification checklist" >}}
- `show ip mroute` on DIST-1 only shows groups A, B, C — D and E don't appear (no downstream subscriber)
- `show ip igmp groups` on each switch shows exactly the 2 groups that host joined
- Kill CORE-R → watch PIM reconverge
- Add a sixth host joining any group → watch the multicast tree extend to a new branch
{{< /details >}}
