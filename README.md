# Scout-Yield-Savings-Simulation-Module

This Python script simulates the operation of a hybrid energy system consisting of a solar PV array, battery storage, and grid backup. It calculates how energy flows are managed — including PV generation, battery charge/discharge, grid use, and total savings.

---

### Purpose

To determine how much of the energy demand is satisfied by:

* Solar PV directly
* Battery discharge
* The utility grid
  And to calculate total **energy savings** as the amount of consumption not drawn from the grid.

---

### System Assumptions

* The load profile (`consumption_kw`) and PV availability (`pv_percent`) are known at specific timestamps.
* The battery has a nominal capacity and is limited by a maximum depth of discharge (DOD).
* The time intervals between input timestamps are variable and used to calculate energy in kWh.

---

### Calculations

For every row in the data:

#### 1. **Time Interval (minutes)**

Calculated as:

```
duration_minutes = current_time - previous_time
```

If it's the first row, a default of 1 minute is used.

---

#### 2. **Energy Demand**

Convert kW load to energy in kWh for the interval:

```
consumption_kwh = consumption_kw * (duration_minutes / 60)
```

---

#### 3. **PV Generation**

Calculate PV energy generated during the same interval:

```
pv_kw_actual = pv_percent * pv_kw * pv_loss_factor
pv_kwh = pv_kw_actual * (duration_minutes / 60)
```

* `pv_percent`: fraction of available PV power (0–1)
* `pv_kw`: total installed PV system capacity
* `pv_loss_factor`: derating factor for panel losses (e.g., dirt, heat)

---

#### 4. **PV vs Load Handling**

Determine how much load is met by PV:

```
if pv_kwh >= consumption_kwh:
    excess_pv = pv_kwh - consumption_kwh
    grid_after_pv = 0
else:
    excess_pv = 0
    grid_after_pv = consumption_kwh - pv_kwh
```

---

#### 5. **Battery Discharge Logic**

If load remains after PV, discharge the battery:

```
available_discharge = SOC - battery_min_kwh
discharge = min(available_discharge, grid_after_pv)
```

* `SOC`: current state of charge (kWh)
* `battery_min_kwh`: 0 (0% SOC)
* `battery_max_kwh`: nominal capacity \* DOD

---

#### 6. **Battery Charge Logic**

If PV excess exists, charge the battery:

```
available_charge = battery_max_kwh - SOC
charge = min(available_charge, excess_pv)
```

---

#### 7. **SOC Update**

```
SOC = SOC - discharge + charge
```

---

#### 8. **Grid Usage**

```
grid_kwh = grid_after_pv - discharge
```

---

#### 9. **Savings Calculation**

```
savings_kwh = consumption_kwh - grid_kwh
```

This gives the total amount of energy not taken from the grid.

---

### Input Format

Each row in `data` should have:

```python
{
    "timestamp": "HH:MM",        # Time of the reading
    "pv_percent": float,         # PV output as percentage (0.0–1.0)
    "consumption_kw": float      # Power draw at that time in kW
}
```

---

### Sample Output

```
   Time  Duration (min)  Consumption (kWh)  PV (kWh)  Excess PV (kWh)  ...
 10:00          600.0            1427.3      216.0             0.0     ...
 12:00          120.0             301.5      256.0             0.0     ...
```

Each row includes:

* Load and PV energy
* Grid shortfall after PV
* Battery charge/discharge
* Grid energy used
* Total savings in kWh

---

### System Parameters

You can adjust:

* `battery_nominal_kwh`: rated capacity (e.g. 290.816)
* `battery_dod`: max depth of discharge (e.g. 0.8 for 80%)
* `pv_kw`: installed PV capacity (e.g. 320)
* `pv_loss_factor`: efficiency derating (e.g. 0.75)

---

### Dependencies

```bash
pip install pandas
```

---


