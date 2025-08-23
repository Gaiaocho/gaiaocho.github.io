# BLE Basics using Zephyr and NRF52840 

Bluetooth Low Energy, 
Is one of the most common communication standards not least due to the smartphone market 
and how far it has propagated, there are billions of BLE enabled applications in the market. 
[This is not out first BLE encounter](https://gaiaochos.com/2024/08/29/Demystifying-Bluetooth-Low-Energy-with-Zephyr.html)

What can one build on that!

BLE mostly works by having a small low powered device (earbuds or sensors) commnicate with a more 
capable one (smart phone, central heating). Here the small low powered devices would be `Broadcasters`
and the capable one woud be `Observer`. Consider a warehouse full of small sensors speaking to a central 
unit that may `(Central)` or may not `(Observer)` have to send data back.

### GAP
The`Observer`, `Broadcaster` and `Central` are considered General Access Profiles `GAP` and this is the layer 
in BLE that specifies how devices discover and interact with each other. 
We will also encounter the `Peripheral` role. 

Lets consider a smart light in a system, what it needs to announce is that it is a light and that it provides a 
way to control whether it is on or off. This process is called advertising where a component will send out periodic 
packages of data about is address, the services it offer and whether it allow for a `Central` to connect to it or 
not. `Central` devices in turn will scan for advertisments of a certain kinds and parses, IDs of these advertisements and 
may establish a connectin, and control thed device. 

### GATT
After the Advertisement and Connection phase of BLE, governed by GAP. We have a formalized way of accessing peripheral 
devices from central devices this way is known and Generic Attribute Profile `GATT`, it defines which services BLE devices 
offer, using the Attribute Protocal `ATT` it defines how data is structurted using `services` and `characteristics`. Services 
and characteristics are saved in peropherals as a look up table known as an `Attribute Table` with each entry in this table 
having a Unique Universal Identifier `UUID`, permissions and callbacks, there is a generic list of [ids](https://www.bluetooth.com/specifications/assigned-numbers/) or you can come up with your won 128-bit addresses. 

`GATT` and `GAP` use different terms to refer to compones in GATT our light is known as a Server and the central is known as 
client. I.E the light can service requests from the phone on whether to turn on or off.

consider the following Light Attribute table. 

```
service: {
    name: "Light Service", 
    ID: "f6a8b8e0-2b4a-49c4-bb43-8625590bd7f8", 
    characteristic : {
        name: "Light State", 
        caps: read/write, 
        ID: "2ab9ec03-9aa0-42df-991b-3b1853bda1a7"
    }
}
```

Explore the [BLE Package format in detail](https://novelbits.io/deep-dive-ble-packets-events/)

---

## Hands-on: Let's explore our [Custom BLE Service](https://github.com/Gaiaocho/zephyr_custom_ble_service)

Setup the NRF52840, make it ready for flashing by pressing the reset button.
Note you need the tools described [here to make everything work](https://docs.zephyrproject.org/latest/boards/nordic/nrf52840dongle/doc/index.html)
 
Get some generic BLE Utility application so we can connect using a phone.

Got through the custom service noting the folliowing:
   - Add a custom 128-bit service UUID.
   Note that the encoding is done by itself, since it ins needed for service creation and advertisement.
   - Add a new characteristic (e.g., "dummy sensor value").
   - Verify read/write operations from the phone.

Note how service/characteristic definitions are structured.

---

## Connection Parameters & PHY Modes 

### Key concepts:
  - Connection interval, slave latency, supervision timeout.
  - PHY options: 1M, 2M, Coded.
  #### Experiment:
  - Change connection interval (e.g., 50 ms → 500 ms).
  - Observe responsiveness vs. current consumption (if tools are available).
  - Note trade-offs between throughput, range, and power.

---

## Bonding & Pairing
- Learn the difference between pairing and bonding.
- Enable bonding in the example (if time permits).
- Test: pair the phone, disconnect, then reconnect to confirm persistence.

---

## Wrap-up
How would you design BLE communications for low power in a wearable? 

**Wrap-up:**
- Summarize:
  - How to add a service/characteristic.
  - Which parameters control power consumption.
  - One optimization idea for a real-world wearable.

