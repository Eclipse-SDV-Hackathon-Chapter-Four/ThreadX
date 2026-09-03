<img width="469" height="162" alt="ThreadX-Color" src="https://github.com/user-attachments/assets/906e065f-d601-43f6-a33f-e94816fd7b51" />

# Using Eclipse ThreadX at the SDV Hackathon
Instructions to leverage Eclipse ThreadX and our IoT Evaluation kits in your mad dash to victory!

By the way, if you need assistance with ThreadX, feel free to ask Frédéric Desbiens! Frédéric, the project lead for Eclipse ThreadX, is one of the hack coaches and will be onsite for the whole duration of the event.

## What is Eclipse ThreadX?

[Eclipse ThreadX](https://threadx.io/releases/6.5.1/home/main/index.html) is an open source real-time operating system (RTOS) designed for deeply embedded and MCU-based systems.

It provides the core facilities you would expect from an RTOS, including:

* threads and priority-based scheduling,
* synchronization,
* queues and message passing,
* timers,
* event flags,
* interrupt handling,
* memory management.

The wider Eclipse ThreadX platform also includes components such as **NetX Duo**, an IPv4/IPv6 networking stack for embedded devices.

For this hackathon, ThreadX gives you something particularly useful: a way to bring a **real embedded computing node** into your Software-Defined Vehicle architecture.

Several **MXChip AZ3166** development boards will be available on site. They combine an STM32 microcontroller with Wi-Fi, an on-board temperature and humidity sensor, accelerometer, magnetometer, buttons, LEDs, an OLED display, and other peripherals. The supplied ThreadX samples already demonstrate board initialization, networking, sensor telemetry, and MQTT communication.

> **Important: There is no standalone ThreadX challenge at this hackathon.**
>
> ThreadX is a technology you can choose to use **inside either of the two main challenges**.
>
> You do not have to use ThreadX to complete a challenge. But if you want to push your solution beyond simulation and explore what happens when SDV software meets a real embedded device, grab an AZ3166.

And don't feel limited to the examples below.

A temperature sensor is the obvious use for the board.

It is definitely **not the only one**.

---

# Start Here

Two resources should cover most of what you need.

## AZ3166 examples

**Eclipse ThreadX SampleX — MXChip AZ3166**

https://github.com/eclipse-threadx/samplex/tree/main/MXChip/AZ3166

This should be your first stop if you want to put ThreadX code onto one of the hackathon boards.

The project already provides several useful application configurations:

* `starter` — board initialization and Wi-Fi connectivity;
* `telemetry` — access to the on-board sensors;
* `mqtt` — telemetry plus MQTT publishing and subscribing;
* `arcade` — examples using more of the board's UI and peripheral capabilities.

Build and deployment scripts are provided for Windows, Linux, and macOS, and ThreadX and NetX Duo are included as dependencies.

If you simply want sensor data, start with **`telemetry`**.

If you need to get that data onto the network, look at **`mqtt`** next.

Then start modifying things.

## Official documentation

**Eclipse ThreadX 6.5.1 Documentation**

https://threadx.io/releases/6.5.1/home/main/index.html

Use the documentation when you want to understand ThreadX itself rather than just modify a sample.

In particular, look at ThreadX services for:

* threads,
* timers,
* queues,
* event flags,
* mutexes and semaphores,
* memory management.

Also look at **NetX Duo** when you start experimenting with networking.

---

# Think of the AZ3166 as a Tiny ECU

The AZ3166 is a development board.

For the hackathon, we encourage you to **pretend it is an ECU**.

Ask yourself:

> What part of my vehicle architecture could reasonably live on a small embedded controller?

It might be:

* a sensor ECU,
* an actuator controller,
* a gateway,
* a health monitor,
* a warning device,
* a protocol adapter,
* an edge-processing node,
* a fault-injection target,
* or something we did not think of.

The examples below are starting points, not requirements.

---

# Challenge 1: Hack to the Future

In **Hack to the Future**, the central question is whether your Guardian Loop can move from virtual components toward real embedded hardware **without changing its business logic**.

The challenge explicitly anticipates replacing a software temperature source with an **AZ3166 running Eclipse ThreadX**, while the Guardian continues to use the same logical service interface.

That is the easiest place to begin.

## Idea 1 — Build a real temperature-sensor ECU

Start with:

```text
Virtual Temperature Sensor
          |
          v
     Guardian Loop
```

Then replace it:

```text
AZ3166
+ ThreadX
+ HTS221 temperature sensor
          |
          v
 Communication / Adapter
          |
          v
     Guardian Loop
```

Your ThreadX application can:

1. acquire the temperature periodically;
2. timestamp or sequence each reading;
3. expose sensor health;
4. transmit the data;
5. let an adapter translate it into the service interface used by Guardian.

The Guardian should not know whether the temperature came from Python, ThreadX, or something else.

That is the portability experiment.

The supplied challenge already identifies replacing the virtual sensor with an AZ3166 + ThreadX endpoint as a hardware progression step.

---

# But Why Stop There?

Once you have ThreadX running, consider doing something more adventurous.

## Idea 2 — Make the AZ3166 a multi-sensor ECU

Instead of publishing only temperature, use several of its peripherals.

For example:

```text
AZ3166 / ThreadX
   |
   +-- Temperature
   +-- Humidity
   +-- Acceleration
   +-- Motion indication
   +-- Device health
   |
   v
Sensor Adapter
   |
   v
Guardian / Vehicle Services
```

You could use the accelerometer as a surrogate for another vehicle signal or experiment with a richer sensor-health model.

The goal is not necessarily to create realistic production sensing.

The goal is to explore what happens when **multiple pieces of physical information originate on an embedded ECU**.

### Guidance

Keep raw hardware handling inside the board or its adapter.

Publish meaningful information such as:

```text
CabinTemperature
SensorHealth
MotionDetected
```

rather than exposing low-level I2C or GPIO operations to Guardian.

---

## Idea 3 — Turn the board into a physical warning device

The AZ3166 has an OLED and LEDs.

Why not make it part of the mitigation path?

For example:

```text
Guardian Loop
      |
      | Warning RPC / Event
      v
uProtocol / Adapter
      |
      v
AZ3166 + ThreadX
      |
      +--> OLED: "CHILD WARNING"
      +--> LED: WARNING
```

Now your architecture contains both:

* a physical sensing endpoint;
* and a physical warning endpoint.

You could even use **two AZ3166 boards**.

```text
AZ3166 #1
Sensor ECU
    |
    v
Guardian
    |
    v
AZ3166 #2
Warning ECU
```

That begins to look much more like a distributed embedded vehicle system.

---

## Idea 4 — Move some preprocessing to the edge

A real ECU often does more than forward raw ADC values.

Your ThreadX endpoint could calculate:

* moving averages,
* rate of temperature increase,
* sample freshness,
* sensor-health status,
* simple plausibility information.

For example:

```text
temperature_celsius: 39.4
temperature_rate: +1.2 C/min
sample_age_ms: 12
sensor_status: OK
```

The Guardian can still own the **hazard decision**.

The embedded ECU simply provides richer information.

### Be careful with the boundary

Avoid quietly moving your entire Guardian algorithm onto the AZ3166.

For this challenge, the interesting result is still that the **Guardian remains portable**.

Think:

```text
Embedded ECU:
"What am I observing?"

Guardian:
"What does that mean for the vehicle?"
```

---

## Idea 5 — Make ThreadX a communication gateway

You don't necessarily need to run every SDV protocol directly on the MCU.

The supplied samples already give you NetX Duo and MQTT foundations.

A perfectly legitimate experimental architecture might be:

```text
AZ3166 / ThreadX
      |
      | MQTT
      v
Linux Gateway
      |
      | uProtocol
      v
Guardian
```

Now ask a more interesting question:

> Can I replace this gateway or transport later without changing Guardian?

That is very much in the spirit of the challenge.

---

## Idea 6 — Hot-swap two physical implementations

If you have access to multiple boards, try treating them as alternative implementations of the same service.

```text
Configuration A

AZ3166 #1
Temperature Service
      |
      v
Guardian
```

Then:

```text
Configuration B

AZ3166 #2
Temperature Service
      |
      v
Guardian
```

Configure them differently.

Maybe one publishes raw temperature while another performs filtering.

If both satisfy the same service contract, Guardian should not care.

Even better: use **openDuT** to manage the transition.

The challenge is specifically designed around changing the test environment without changing the Guardian application.

---

# Challenge 2: Doctor Whodunit

In **Doctor Whodunit**, the question changes.

You are no longer asking only:

> Can I replace software with hardware?

You are asking:

> What happens when that hardware lies, freezes, becomes slow, disappears, or behaves strangely — and can I prove what happened?

That makes a ThreadX device very interesting.

Instead of being just a physical sensor, the AZ3166 can become a **controllable, observable, failure-prone embedded ECU**.

---

## Idea 1 — Build a faultable temperature ECU

Start with the obvious implementation:

```text
AZ3166
+ ThreadX
+ Temperature Source
       |
       v
Battery Thermal Guardian
```

Then add deliberate fault modes.

For example:

```text
NORMAL
STUCK_VALUE
OUT_OF_RANGE
NO_DATA
SLOW_DATA
INTERMITTENT_DATA
```

A run could look like:

```text
37.2 C
38.1 C
39.0 C

Inject STUCK_VALUE

39.0 C
39.0 C
39.0 C
39.0 C

Guardian:
STALE / STUCK SENSOR DETECTED
```

Now you have created an actual embedded participant in your fault campaign.

---

## Idea 2 — Break the RTOS behavior, not only the sensor value

ThreadX itself gives you interesting things to experiment with.

Suppose your application has:

```text
Sensor Thread
Communication Thread
Health Thread
```

Instead of injecting only bad temperature data, deliberately perturb the software.

For example:

* suspend the sensor thread;
* delay the communication thread;
* overwhelm a message queue;
* create resource contention;
* intentionally miss a periodic deadline;
* stop updating a heartbeat while other processing continues.

Now the Guardian may see very different symptoms.

```text
Sensor failure
        !=
Communication failure
        !=
Whole ECU failure
```

That distinction is exactly the sort of evidence Doctor Whodunit can explore.

### Guidance

Make these failures **intentional and controllable**.

Do not rely on random crashes.

Ideally your test harness should be able to say:

```text
inject fault: SENSOR_THREAD_STALL
```

and get the same result every time.

---

## Idea 3 — Build an embedded health supervisor

Give the ThreadX system its own internal supervision.

For example:

```text
Temperature Thread ----+
                       |
Network Thread --------+--> Health Monitor
                       |
Sensor Driver ---------+
```

The health monitor could report:

```text
ecu_alive: true
sensor_alive: true
network_alive: true
last_sample_age_ms: 34
fault_flags: 0x00
```

Now consider a fault:

```text
Sensor thread stops
```

The board itself might detect:

```text
sensor_alive = false
```

before the Guardian notices that the temperature stream has become stale.

That gives you **two layers of evidence**:

```text
Embedded diagnosis
        +
Vehicle-level diagnosis
```

Correlating those two views can make a very compelling Doctor Whodunit demo.

---

## Idea 4 — Create a tiny "black box"

Use the MCU as an evidence producer.

Keep a small circular history of significant events:

```text
T+0000 ECU START
T+1050 TEMP 37.1
T+2050 TEMP 37.8
T+3050 FAULT STUCK_VALUE
T+4050 TEMP 37.8
T+5050 TEMP 37.8
T+5210 GUARDIAN_WARNING_RX
```

After a test campaign, export or print the history.

You now have evidence from the perspective of the embedded device itself.

Correlate that with:

* Guardian logs,
* uProtocol events,
* OpenSOVD diagnostics,
* Ankaios state,
* openDuT test execution.

That starts to resemble a distributed safety-evidence timeline rather than a collection of unrelated logs.

---

## Idea 5 — Play with timestamps and sequence numbers

Some of the hardest failures are not values that are obviously wrong.

Try sending apparently reasonable data with subtly broken metadata.

For example:

```text
seq 100  temp 39.1  timestamp 10:00:01
seq 101  temp 39.3  timestamp 10:00:02
seq 103  temp 39.7  timestamp 10:00:04
seq 102  temp 39.5  timestamp 10:00:03
```

What should Guardian conclude?

Try:

* duplicate sequence numbers;
* missing messages;
* old timestamps;
* future timestamps;
* reordered messages;
* long pauses followed by bursts.

This lets the physical ECU participate directly in the challenge's communication- and signal-level fault scenarios.

---

## Idea 6 — Make the board both the victim and the detective

A fun architecture is to have the board report its own internal health while another component observes its external behavior.

```text
             +--> Temperature Events
AZ3166 ------|
             +--> Heartbeat / Health Events
```

Then inject a failure.

Suppose temperature events disappear but heartbeat remains alive.

That tells a very different story from:

```text
Temperature gone
Heartbeat gone
```

Your evidence system can now answer a much better question than:

> "Did I stop receiving temperature?"

It can ask:

> "Did the sensor function fail, did communication fail, or did the entire ECU disappear?"

Now you are doing **Doctor Whodunit**.

---

## Idea 7 — Use two boards to create a distributed failure

With two AZ3166 boards:

```text
AZ3166 #1
Sensor ECU
     |
     v
Vehicle Network
     |
     v
AZ3166 #2
Monitor / Warning ECU
```

Try failures at different locations.

For example:

```text
Case A:
Sensor ECU freezes.

Case B:
Sensor ECU works,
but communication is interrupted.

Case C:
Sensor data is delivered,
but values are implausible.
```

Can the rest of your system distinguish them?

Can your evidence collector reconstruct what happened?

That is a much richer experiment than simply changing a number in a Python simulator.

---

## Idea 8 — Give fault injection a physical interface

The AZ3166 buttons and display can be useful during development.

For example:

```text
Button A -> STUCK SENSOR
Button B -> SENSOR DROPOUT
OLED     -> Current fault mode
LED      -> Device health
```

This makes faults very easy to demonstrate interactively.

For your final evidence campaign, though, try to make the same faults remotely controllable as well.

Ideally:

```text
openDuT / Test Controller
          |
          v
     Fault Command
          |
          v
    AZ3166 / ThreadX
          |
          v
Activate SENSOR_STUCK
```

Manual buttons are great for demos.

Automated fault injection is better for repeatability.

---

# Some Principles to Keep You Out of Trouble

Experiment aggressively, but keep a few architectural rules.

## Keep the service contract stable

Whether your implementation is:

```text
Python simulator
```

or:

```text
AZ3166 + ThreadX
```

the rest of the system should ideally see the same logical interface.

That is especially important in **Hack to the Future**, where the Guardian is explicitly meant to remain isolated from hardware-specific code.

---

## Instrument your embedded application

When you move from simulation to hardware, observability becomes extremely valuable.

Consider including:

```text
timestamp
sequence_number
boot_id
sensor_status
heartbeat
fault_mode
```

in your telemetry or diagnostic information.

You will thank yourself when something stops working.

---

## Separate the data plane from the test plane

Your normal application might produce:

```text
TemperatureEvent
```

while a separate test interface accepts:

```text
InjectFault(STUCK_SENSOR)
```

Keeping those concerns separate makes experiments easier to automate and explain.

---

## Start simple — then get weird

Do not spend half the hackathon implementing your first I2C read.

First:

```text
Build sample
Flash board
Read sensor
Send data
```

Then extend it.

Once the basic path works, that is the time to try:

* multiple threads,
* health supervision,
* distributed ECUs,
* fault injection,
* edge processing,
* alternative transports,
* hardware swapping,
* embedded diagnostics,
* physical warning devices.

---

# One Board, Two Different Mindsets

The same ThreadX/AZ3166 platform can play very different roles.

### Hack to the Future

```text
ThreadX = another implementation of a vehicle capability

Question:

Can I swap this physical implementation
into my architecture without changing
the Guardian?
```

### Doctor Whodunit

```text
ThreadX = an observable, controllable,
failure-prone embedded ECU

Question:

Can I break this component in interesting
ways and prove that the rest of the system
understands what happened?
```

Both are excellent uses of ThreadX.

Again, **there is no separate ThreadX challenge**.

ThreadX is here because real SDV systems do not consist only of containers and Linux services. Somewhere, eventually, software meets sensors, actuators, deadlines, interrupts, physical networks, and tiny computers.

The AZ3166 gives you an opportunity to explore that boundary.

The detailed examples above are deliberately achievable.

But they are not a specification.

If your team looks at the board and thinks:

> "What if we used ThreadX to..."

that is probably a conversation worth having.
