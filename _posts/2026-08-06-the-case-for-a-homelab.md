---
title: "The Case for a Homelab"
description: Dual-WAN failover, filtered DNS, and an automation stack that rebuilds from one script. Why production discipline belongs at home, and in small offices.
date: 2026-08-06
tags: [homelab, architecture, security]
---

My home network runs two fibre connections from competing ISPs, load-balanced
50/50 by a MikroTik router that also hosts the network's DNS filter in a
container. Behind it, a Raspberry Pi runs the automation stack: Home
Assistant, an MQTT broker, a container management plane. The whole thing
rebuilds from a git repository and one bootstrap script.

That is more infrastructure than a household strictly needs. It is also the
cheapest architecture education I have ever given myself. The argument of
this post, the first in a series, is that a system like this belongs in more
homes and small offices than you might expect: not as a hobby, but as a
working scale model of the disciplines that matter everywhere else.

<!--more-->

## What actually runs here

```
     Airtel fibre              Jio fibre
          │                        │
    ether1-airtel            ether2-jio
          └───────────┬────────────┘
              PCC load balancing
           50/50, connection-sticky
                      │
             MikroTik L009UiGS
              LAN 10.1.0.1/24
                      │
       ┌──────────────┼──────────────┐
       │              │              │
  LAN bridge     Pi-hole DNS    Raspberry Pi
  wired and      (container     ├ Home Assistant
  wireless        on the        ├ Mosquitto MQTT
  clients         router)       └ Portainer
```

Three planes, each with one job.

The edge is the router, with both fibre links terminated on it. New
connections are split evenly across the ISPs by per-connection
classification, so a video call stays pinned to one link while bulk traffic
spreads across both. Failover uses recursive routing: each link is
continuously health-checked against well-known internet addresses, and when
the checks fail, that ISP's routes are withdrawn and everything shifts to the
surviving link.

```
 check addresses      virtual next-hop     ISP gateway
 1.0.0.1, 8.8.8.8 ──▶   10.1.1.1  ──ping──▶  Airtel
 8.8.4.4, 1.1.1.1 ──▶   10.2.2.2  ──ping──▶  Jio

 pings fail → virtual hop unreachable → route withdrawn
            → traffic moves to the surviving link
```

Nobody in the house notices an ISP outage. I read about it afterwards, in
the email the router sends when an interface changes state.

DNS for every device flows through Pi-hole, so advertising and tracker
domains are filtered at the network layer with no per-device configuration.
The filter runs as a container on the router itself: the appliance at the
edge is also a small container host.

The automation plane is a Raspberry Pi running Home Assistant, a Mosquitto
MQTT broker and Portainer under Docker. Air conditioners, fans, lights and
heaters publish state over MQTT; Home Assistant schedules them and meters
energy consumption per room.

The toolset, by role:

- **RouterOS / WinBox** — routing, firewall and management on the MikroTik
  edge; the entire configuration is scriptable and exportable.
- **Pi-hole** — network-wide advertising and tracker blocking at the DNS
  layer, for every device with no client configuration.
- **Unbound** — recursive DNS resolution behind Pi-hole, answering from the
  root servers instead of handing the query log to a public resolver.
- **Home Assistant** — the automation brain: schedules, scenes and per-room
  energy metering.
- **Mosquitto** — the MQTT broker every sensor and switch speaks through.
- **Nginx Proxy Manager** — one front door for the service UIs, with proper
  names and TLS instead of a notebook of IP-and-port pairs.
- **Portainer** — visibility and management for the container stack.

The part I care most about is invisible in the diagram: the entire stack is
declared in a git repository. A `.env` file holds the instance-specific
values, a bootstrap script installs Docker, generates the secrets and lays
out the directory tree, and `docker compose up` does the rest. The Pi is
replaceable hardware, not a snowflake.

## Why a household needs an architecture

A homelab is usually defended as a learning environment. That undersells it.
It is a production system with real users, and the SLOs are enforced by your
family. When the internet drops during a school exam or the lights refuse an
automation at midnight, no framing of "it's just a lab" survives contact
with the user base.

That pressure is precisely what makes it valuable. Every discipline that
matters at enterprise scale has an honest scale model here:

- **Redundancy** — two commodity fibre links and deliberate routing buy the
  availability that one "business grade" link promises.
- **Failure domains** — the router, the DNS path and the automation plane
  fail independently, and the design has to answer for each.
- **Reproducibility** — if the Pi dies, recovery is a script and a restore,
  not an afternoon of archaeology.
- **Constraint budgeting** — the router has 512 MB of RAM shared with its
  DNS container. Capacity planning is not optional at any scale.

The blast radius is a household. The lessons are not. A homelab is the
cheapest place I know to learn expensive lessons.

## The same pattern, small-office sized

Nothing above requires enterprise budget. Two consumer fibre plans and a
router in the price range of a mid-tier phone deliver load-balanced,
self-healing internet. For a ten-person office, a clinic or a studio, that
is business continuity at consumer prices: an ISP outage becomes a log line
instead of a lost morning.

The rest of the stack translates just as directly. Network-wide DNS
filtering removes a class of malware and advertising without touching a
single client device. Local automation handles scheduling and energy
metering without a cloud subscription or a vendor dependency. And because
the configuration lives in git with secrets kept out, the setup survives
the failure of any single box, including the person who built it being
unavailable, which is the failure mode small offices plan for least.

## Security has to be designed in, then audited

A homelab is also where security stops being abstract. The perimeter here is
default-drop: nothing reaches the router from the WAN side unless a rule
explicitly allows it, and management services are reachable only from the
LAN. DNS is a control point, not just a convenience. The repository
publishes structure, never credentials: secrets are generated at bootstrap
and excluded from version control.

Then I did what I would do to any production system: exported the router's
configuration and ran a cold findings review against it. Twenty findings,
three of them critical. A container password sitting in plain text in the
config export. Management services I had never explicitly restricted,
because the firewall in front of them made it feel unnecessary. A VPN server
stub configured years ago, unused, and quietly widening the attack surface.

Each finding is a small embarrassment and a better teacher than any
checklist. Defence-in-depth exists because the layer you trust will one day
be misconfigured. Configuration exports are secrets and deserve the same
handling. And an unused service is not neutral; it is surface. Auditing your
own work with the same coldness you would apply to a client's is a
discipline, and a homelab gives you somewhere consequential to practise it.

## Where this series goes

This post is the overview. The ones that follow will each take one plane
apart properly: the dual-WAN design (per-connection classification, recursive
routes and the mangle table that makes failover boring), the reproducible
Raspberry Pi stack (the `.env` contract, the bootstrap script, migration and
backup), and the hardening pass that closes out the audit findings.

None of it is exotic hardware or heroic configuration. That is the point.
The homelab's real output is not uptime; it is judgment, practised where the
stakes are survivable.
