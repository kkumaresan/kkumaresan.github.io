---
title: "A Soldering Iron and a Message Bus"
description: Boards I built by hand, C++ firmware I wrote to drive them, and a scheduler that runs every ten minutes. What the automation tier actually does, and what it taught me about designing interfaces you cannot redeploy.
date: 2026-08-11
tags: [homelab, iot, mqtt, architecture]
series: The Case for a Homelab
part: 2
---

[Part one](/writing/the-case-for-a-homelab/) argued that a homelab is a
production system with real users. This is the part below that argument: the
devices themselves. A Raspberry Pi in Chennai runs four containers — Home
Assistant, Mosquitto, Portainer and a reverse SSH tunnel — and everything
interesting hangs off the second of those. Most of what publishes to that
broker is not a product. It is a board I built at the dining table, flashed
and screwed into a wall. None of it can be redeployed from a laptop. Some of
it is glued behind an air conditioner's grille, in a flat where the ambient
temperature sits above thirty degrees for eight months of the year.

That constraint shapes every decision that follows.

<!--more-->

## What is actually on the walls

The device tier is deliberately boring, and mostly two chips.

<figure class="figure-wide">
  <img src="{{ '/assets/img/fig-02-device-layer.png' | relative_url }}"
       width="1600" height="1600" loading="lazy" decoding="async"
       alt="Block diagram of the device layer. Home Assistant, the automation plane, runs a scheduler that re-asserts the desired state of every room on a fixed interval. Below it an MQTT broker carries paired state and command topics under a zone/device/attribute contract, plus a last-will availability topic. Hanging off the broker: a hand-built ESP8266 or ESP32 IR controller for the air conditioners, running C++ firmware that sends full-state frames; a hand-assembled relay board flashed with Tasmota for fans, heaters and lighting; and door, PIR, BME280, HTU21D and current-clamp sensors.">
  <figcaption>Fig. 02 — Device layer. Everything below the broker is a dumb
  appliance with a small computer taped to it.</figcaption>
</figure>

The air conditioner controllers are mine end to end: an ESP — 8266 on the
older boards, ESP32 on everything I have built since — an IR LED, a driver
transistor and a temperature sensor, on a board I assembled, running
firmware I wrote in C++. More on that below, because it is the part of this
system I am least willing to hand-wave.

The fan and heater controllers are hand-built too — relay boards I put
together and flashed with Tasmota, where stock firmware was genuinely enough
and writing my own would have been ego rather than engineering. Off-the-shelf
hardware fills the rest: door and motion sensors watching the main entrance,
BME280 and HTU21D sensors for temperature, humidity and pressure, and
current transformers clamped around the heavy circuits. All of it lands on
the same bus, in the same shape, whatever it is underneath.

That split is the honest summary of the build. Write the firmware where the
problem is genuinely unsolved; flash someone else's where it isn't. Nothing
here is a smart appliance. Every one of these is a dumb appliance with a
small computer I built taped to it, and that is the point: the intelligence
is central and version-controlled, and the edge is replaceable for the price
of a board and an evening.

## The firmware I had to write

Air conditioners were the part nobody could sell me a solution for, so I
wrote one in C++ against Espressif's own SDK rather than the Arduino layer
over it — the ESP8266 RTOS SDK on the early boards, ESP-IDF proper on the
ESP32s I have built since. Same idioms, same FreeRTOS underneath, so the port
cost me an afternoon. That choice costs you a weekend of build system and
menuconfig before a single LED blinks, and buys you a device that behaves
predictably for years.

The problem is that an AC remote speaks a protocol its manufacturer does not
publish. You point an IR receiver at the remote, capture the carrier and
stare at pulse timings until structure appears: a header burst, a bit
encoding where a one and a zero differ only in the length of the gap after an
identical mark, a payload and a checksum at the end that has to be derived
by pressing buttons in sequence and watching which byte moves. Then you emit
it back — a timing array shifted out of a GPIO pin at 38 kHz. Getting the
checksum wrong on the first brand cost me an evening of an AC that beeped and
did nothing. The second brand took an hour, because by then I understood the
shape of the problem.

The design constraint that mattered most only became obvious mid-way: these
remotes do not send deltas. Press "temperature up" and the remote transmits
the *entire* state — mode, setpoint, fan speed, swing, timer — in one frame.
The air conditioner is a receiver with no memory of the conversation, which
means the board on the wall has to be the thing that remembers. So the
firmware carries a state struct, mutates one field when a command arrives,
and retransmits the whole frame every time. That single fact shapes
everything upstream: it is *why* there is a separate state topic and command
topic, and why the scheduler can safely reassert the same state every ten
minutes without confusing the unit.

The rest of the firmware is unglamorous and matters just as much, and this is
where working close to the SDK pays for itself. Separate FreeRTOS tasks for
the IR transmit path, the network client and the sensor sampling, talking
through queues, so nothing blocks anything else — the IR frame has
microsecond-sensitive timing, and a network stall in the middle of a
transmission produces a command the air conditioner silently ignores. WiFi
and broker reconnection with backoff, since a board that reboots at 2 AM must
rejoin and re-announce itself without anyone noticing. A last-will message
registered at connect time, so the broker reports the board's death if it
cannot. Static allocation and no clever abstractions, because the thing has
to run for months untouched behind a grille in forty-degree heat, and heap
fragmentation on a device you cannot reach is not a bug you get to fix.

Writing it myself is also what made the rest of the design possible. A device
whose firmware I control can publish exactly the topics my architecture
wants, in the shape my architecture wants, rather than forcing the platform
to adapt to whatever a vendor decided to emit.

## The topic scheme is the architecture

If you take one thing from this post: on an IoT network, your MQTT topic
scheme *is* your API, and you will live with it far longer than you expect.

The shape worth copying is this one:

```
<prefix>/<zone>/<device>/<attribute>/<direction>

  …/<device>/mode/state         →  what the unit is doing
  …/<device>/mode/command       →  what I have asked it to do
  …/<device>/temperature/{…}    →  one pair per attribute
  …/<zone>/ambient/temperature  →  what the room actually reads
  …/<device>/availability       →  online / offline, via last will
```

Three properties earn their keep. Every controllable attribute is split into
a state topic and a command topic, so the system never confuses "what I asked
for" with "what is true" — the single most valuable line in the whole design.
Ambient temperature comes from an independent sensor rather than from the air
conditioner's own reading, so the control loop is closed against the room,
not against the appliance's optimism. And every device registers a last-will
message with the broker, so when a board drops off the WiFi the broker
announces the death on the device's behalf. Availability is not something the
automation platform has to infer from silence.

The scheme also carries an honest scar, and I would rather describe it than
show it. My production topics use two different top-level prefixes, split
along a boundary that made sense on the evening I introduced it and makes no
sense now, and the house has a third name again inside Home Assistant. Each
choice was locally reasonable. Together they mean I keep a mental translation
table to debug my own network. Renaming would require reflashing boards that
are now behind furniture and inside grilles, so the drift is permanent — the
cheapest possible lesson in why naming conventions are load-bearing
infrastructure, learned at the price of never being able to fix it.

## Scheduling is a control loop, not a rule

The obvious way to automate an air conditioner is a rule: at 22:00, turn on.
I started there. It fails within a week, because rules are edge-triggered and
houses are stateful. A command lost to WiFi contention is lost forever. A
device that reboots at 22:01 never gets the message. Someone turns the unit
off manually and the system never notices.

So the climate system is a loop instead. Every ten minutes — plus immediately
on any control-switch change — an automation calls a scheduler script with a
list of rooms. For each room it reads three schedule slots out of Home
Assistant helpers: a start time, an end time, a target temperature between 18
and 30, and a fan-overlap toggle. Then it asserts the correct state, whatever
the current one happens to be.

```
every 10 min ─→ for each room ─→ inside a window?  → cool @ target
                                  in the overlap?  → AC off, fan on
                                  otherwise        → AC off, fan off
```

The loop is idempotent, so a dropped IR command self-heals on the next tick,
and the worst-case cost of any single failure is ten minutes of wrong
temperature. Three slots per room exist because a Chennai night is three
different problems: the evening pre-cool, the deep-night hold and the
pre-dawn hour when the outside air finally drops below the setpoint.

The fan overlap is my favourite piece of the whole system and the least
technical. When the AC's window closes, the ceiling fan runs on for another
seven minutes — the default, tunable per room — to move the cold air already
in the room instead of paying the compressor to make more. It is one number
in a config file, it is worth real money over a Chennai summer, and it exists
only because someone in the house complained about waking up cold at 3 AM.

## Amps to kilowatt-hours

The energy pipeline is four transformations, each one cheap and each one
auditable.

```
CT clamp → amps (MQTT) → × 220 V → watts → Riemann sum → kWh → utility meter (day/week/month)
```

A clamp on the circuit publishes amperage. A template sensor multiplies by
mains voltage to get power. An integration sensor performs a left Riemann sum
over time to accumulate energy. Utility meters slice that total into daily,
weekly and monthly cycles that reset on their own. Per appliance, per room.

It is worth being clear-eyed about the accuracy. Multiplying amps by a
nominal 220 V assumes a power factor of one, which is false for anything with
a motor in it — which is to say, false for every air conditioner in the
house. The absolute numbers are wrong. The *comparative* numbers are useful,
and comparison is what actually changes behaviour: which room costs the most,
whether the fan overlap paid for itself, what happened to consumption when
the setpoint moved by one degree. Precision I do not have; direction I do.

The staged version adds three-phase whole-home monitoring on R, Y and B,
which is where per-appliance measurement stops being a curiosity and starts
reconciling against the utility bill.

## The part with real stakes

Air conditioners are comfort. Heaters are not: a resistive element left on is
a fire, and everything else in this post is a hobby by comparison.

So the utility scheduler is not a scheduler at all, it is a state machine
with a physical feedback signal. Each appliance is declared with four
references — a manual control switch, an automation-enable switch, the relay
that actually carries current, and a current sensor. It runs a load timer
that caps continuous operation, a reset timer that enforces a cool-down
before the next cycle, and a tolerance threshold on the current reading.

That current sensor is the whole design. Without it, "off" means the system
believes it sent an off command. With it, "off" means the circuit is drawing
under a tenth of an amp. A welded relay, a stuck Tasmota, a command that
never landed — all of them look identical from the software side and
completely different from the clamp. On an overcurrent or a state mismatch,
the system cuts the relay and pushes a notification to the four phones in the
family group. Nobody has to be watching a dashboard.

This is the pattern I would defend most strongly to anyone building the same
thing: for any device that can hurt someone, close the loop with a physical
measurement, not with your own command history.

## What the soldering iron taught the architect

None of this is exotic hardware. It is a handful of cheap boards, a message
bus and a loop that asserts the same thing every ten minutes until the house
agrees. What made it worth building is that every layer punishes a different
kind of sloppiness, and it punishes you at home, where the blast radius is
one flat.

The firmware taught me that a protocol you cannot read is just a timing
diagram you have not been patient enough with. The topic scheme taught me
that an interface outlives the code on both sides of it, and that you should
design it as though you will never be able to change it — because on devices
sealed behind a grille, you cannot. The scheduler taught me that state you
assert beats state you announce. The current clamp taught me that trust in
your own commands is not evidence.

Ten minutes of wrong temperature is a survivable mistake. That is the whole
reason to practise here.
