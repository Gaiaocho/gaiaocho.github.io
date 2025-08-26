# Low Power Networking with BLE


## Goal
Optimize BLE advertising and connection settings for minimal power consumption, 
understand PHY trade-offs, and verify behavior on hardware.

---

## Baseline Power and Network Behavior

#### Current BLE advertising and connection setup.
Our current advertise  starts on start up and is in the `ble_service` function `ble_le_adv_start`
using params defines in `advert[]` data. 
This array constains AD types that can be encoded in advertising data.
 
Note advertising interval, duty cycle these are set to the defaults provided by 
`BT_LE_ADV_CONN_FAST_1`, currently but the following can be changed. 

a) Advertising interval both maximum and minimum, this is the time between consecutive 
advertising effects, this can be set to a long time to lower power usage
b) Peer device this is for directed advertising again if the service is not generally 
consumed this will save a lot, We  can have a whitelist, devices can reconnect after 
power cycles and security tokens can always reused on reset. 
Ideal for when we want bonded, secure and stable connections. 
c) Advertising options
    - Are we connecatable?
    - Are we using our ID address (this is not safe)
    - Low Duty / High duty basically how much time the radio is on or off 
    for connection, advertising and scanning. 


- Baseline Power STATs from PM?
use `CONFIG_PM_STATS` and `CONFIG_PM`, log out info.

```c
struct pm_state_info info;    
pm_state_info_get(&info, PM_STATE_ACTIVE);
printk("Active state: %u ms, %u transitions\n", info.time, info.count);
```

---

## Advertising  & Connection Parameters for Low Power

a) Currently we enable BLE and disable BT Classic
b) We are currently always discoverable and this is a waste
c) We dont set a peer param so we still always advertise to the world we can set 
just one peer device to save power.
d) We can also make a very short advert and perhaps provide more data in the scan 
response this way a `central` can read if they want.


We can move to a more refined advertising scheme using  the `bt_le_adv_param` struct.
in place of using the `BT_LE_ADV_CONN_FAST_1` preset we are using now.

- Zephyr advertising presets:
  - `BT_LE_ADV_CONN`
  - `BT_LE_ADV_CONN_LOW_DUTY`
- Experiment with `BT_LE_ADV_PARAM_INIT()` to set custom intervals:
  - Increase advertising interval (e.g., 500 ms → 2 s).
  - Reduce duty cycle where possible.

``` c
struct bt_le_adv_param adv_params = {
    .options = BT_LE_ADV_OPT_CONNECTABLE,
    .interval_min = 1600,  // 1s (1600*0.625ms)
    .interval_max = 2000,  // 1.25s
    .secondary_phy = BT_GAP_LE_PHY_CODED,
};

err = bt_le_adv_start(&adv_params, advert, ARRAY_SIZE(advert), NULL, 0);
```

In the real world things are not so clear cut, we can move between connection parameter  
presets in order to implement things like `fast mode` and `power savings mode` for this 
we have the API - `bt_conn_le_param_update()` so we can do something like fast connection 
then switch to low power modes. 

---

#### BT 5+ PHY and EXT and Data Rates
- Extended advertisng and PHY modes are features introduced in BT 5 mainly to improve
rance, throughput and flexibility.
1. LE 1M PHY: Legacy 1Mbs, default 
2. LE 2M PHY: 2Mb rate, shorter range, lower energy
3. LE Coded PHY: 1Mb rate, significant range improvement at the cost of throughput

We can also move between phy modes depending on the enviroment and the profile we are 
looking for and this is  enabled by the API
  - `bt_conn_le_phy_update()`
For sensor data (small, infrequent packets), 1M or 2M is usually best for low power.

---

### Extras
- Experment with setting a tx power level and see how low one can push it before we get 
significant attenuation

## Integration and Documentation
- Combine changes:
  - Adjust advertising for low power.
  - Apply optimized connection parameters.
  - Optionally set PHY.
- Verify device still connects reliably.

- Document:
  - Chosen parameters.
  - Trade-offs between responsiveness, range, and energy savings.
  - Future improvements (measuring current with a power profiler).

