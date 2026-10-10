# 🚛 SII Save File Editing Guide for ETS2/ATS

A practical reference for reading and editing `game.sii` save files in ETS2 and ATS. Covers the SII format, common units, and common editing fields for version 1.61.

> **Always back up your save before editing.**

## Links
- [Basic save edit tutorial By LordBenji](https://forum.truckersmp.com/index.php?/topic/44399-save-editing-tutorial-in-depth-video/)
- [SCS Modding Wiki](https://modding.scssoft.com/wiki/Main_Page) - [Documentation/Engine/Units](https://modding.scssoft.com/wiki/Documentation/Engine/Units)

---

<br>

# Table of Contents

1. [File Structure & Overview](#file-structure--overview)
2. [Value Types](#value-types)
3. [Unit Types in Order of Appearance](#unit-types-in-order-of-appearance)
   - [economy](#economy)
   - [bank](#bank)
   - [player](#player)
   - [player_vehicles](#player_vehicles)
   - [trailer](#trailer)
   - [trailer_utilization_log](#trailer_utilization_log)
   - [trailer_utilization_log_entry](#trailer_utilization_log_entry)
   - [trailer_def](#trailer_def)
   - [selected/current job](#selectedcurrent-job)
   - [vehicle](#vehicle)
   - [vehicle accessories](#vehicle-accessories)
   - [truck_profit_logs](#truck_profit_logs)
   - [profit_log_entry](#profit_log_entry)
   - [company](#company)
   - [job_offer_data](#job_offer_data)
4. [And More...](#and-more)
5. [Working with Arrays](#working-with-arrays)
6. [Common Edits Reference](#common-edits-reference)
7. [Regex Reference](#regex-reference)

---

## File Structure & Overview

The `game.sii` file uses the **SII** format with the following structure:

```
SiiNunit
{
    // Unit definitions
    unit_type : unit_name {
        attributes
    }
    
    // More unit definitions...
}
```

## File Characteristics
- **Magic marker:** `SiiNunit` (starts the file)
- **Opening brace:** `{` (line 2)
- **Closing brace:** `}` (end of file)
- **Line endings:** CRLF (`\r\n`)
- **Encoding:** UTF-8 (no BOM)

## Reference System

SCS uses a unique **addressing system** to link related data across the save file:

```
_nameless.XXX.XXX.XXX
```

- `_nameless` = Prefix for nameless units
- `XXX.XXX.XXX` = Unique hash identifiers (changes per save)

## owner_ptr vs link_ptr

**owner_ptr:** Points to a unit defined within the **same SiiNunit** (nameless units like `_nameless.XXX.XXX.XXX`)

**link_ptr:** Points to a **named unit** that is defined elsewhere (full paths like `company.volatile.xxx`)

In save files, you'll typically see:
- `_nameless.XXX.XXX.XXX` format → **owner_ptr** (same SiiNunit)
- `unit_type.name` format → **link_ptr** (named unit elsewhere)

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## Value Types

| Type | Format | Example | Description |
|------|--------|---------|-------------|
| **string** | Double-quoted | `"BQ-217-LH"` | Text strings |
| **float** | Decimal or hex | `1.0` or `&3f800000` | Wear, fuel, ratios |
| **token** | Lowercase alphanumeric + underscore | `flatbed`, `single` | Short identifiers (max 12 chars) |
| **bool** | `true` / `false` | `app_enabled: true` | Boolean values |
| **int2/3/4** | Tuple | `(1, 2)`, `(1, 2, 3)` | 2D/3D/4D integer vectors |
| **s16/s32/s64** | Signed integers | `-15` | Signed integer types |
| **u16/u32/u64** | Unsigned integers | `15` | Unsigned integer types |
| **fixed** | Integer | `10` | Fixed-point integers |
| **placement** | Position + rotation | `(0, 0, 0) (1; 0, 0, 0)` | 3D position + quaternion |
| **quaternion** | w,x,y,z | `(1.0; 0, 0, 0)` | Rotation/orientation |
| **owner_ptr** | Nameless reference | `_nameless.XXX.XXX.XXX` | Points to unit in same SiiNunit |
| **link_ptr** | Named reference | `company.volatile.xxx` | Points to unit elsewhere |
| **null/nil** | `null` or `nil` | `current_job: null` | Empty/undefined values |

## Hex Float Reference

Hex floats use `&` prefix (IEEE-754 format):

| Hex | Decimal | Use Case |
|-----|---------|----------|
| `&00000000` | 0.0 | Empty, zero |
| `&3f800000` | 1.0 | Full (100%) |
| `&3dcccccd` | 0.1 | 10% |
| `&3e800000` | 0.25 | 25% |
| `&3f000000` | 0.5 | 50% |
| `&3f4ccccd` | 0.8 | 80% |
| `&3f666666` | 0.9 | 90% |
| `&40000000` | 2.0 | 200% |

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## Unit Types in Order of Appearance

## economy

**Purpose:** Root container for all game state data

**Description:** The `economy` unit is the root of your save file and contains references to all major game systems including the bank, player, list of companies and garages, amount of experience points and skill level and various game controllers.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `bank` | owner_ptr | Reference to bank unit |
| `player` | owner_ptr | Reference to player unit |
| `companies` | int | Number of companies |
| `companies[i]` | link_ptr | References to company units |
| `garages` | int | Number of garages |
| `garages[i]` | link_ptr | References to garage units |
| `experience_points` | int | Amount of experience points |
| `skill` | int | Level of different skills |

**Example:**
```sii
economy : _nameless.254.03df.8ba8 {
 bank: _nameless.253.df94.3778
 player: _nameless.254.0377.2108
 companies: 1981
 companies[0]: company.volatile.bhv.kristiansand
 companies[1]: company.volatile.ibp.naantali
 ...
 companies[1980]: company.volatile.lkwlog.eindhoven
 garages: 234
 garages[0]: garage.kobenhavn
 ...
 garages[233]: garage.patras
 game_time: 37205
 game_time_secs: &41c0cb4d
 experience_points: 216983
 adr: 63
 long_dist: 6
 heavy: 6
 fragile: 6
 urgent: 6
 mechanical: 6
}
```

**Regex to locate:**
```regex
economy\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Increase xp:** Change `experience_points` to a larger number
- **Set individual skill level:** Change `adr` to `63`, `long_dist` to `6` for max level

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## bank

**Purpose:** Player's bank account and financial data

**Description:** Contains all financial information including bank balance, insurance, loans, and overdraft settings.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `money_account` | int | Bank balance |
| `coinsurance_fixed` | int | Fixed coinsurance amount |
| `coinsurance_ratio` | float | Coinsurance ratio (hex float) |
| `accident_severity` | float | Accident severity multiplier |
| `loans` | int | Number of active loans |
| `loan_limit` | int | Maximum loan limit |
| `overdraft` | bool | Overdraft enabled flag |

**Example:**
```sii
bank : _nameless.253.df94.3778 {
 money_account: 235294705290
 coinsurance_fixed: 1000
 coinsurance_ratio: &3dcccccd
 accident_severity: &40f1a512
 loans: 0
 app_enabled: true
 loan_limit: 500000
 overdraft: false
}
```

**Reference Chain to This Unit:**
```
economy → bank
```

**Regex to locate:**
```regex
bank\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Increase money:** Change `money_account` to a larger number
- **Increase loan limit:** Change `loan_limit` to a larger number

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## player

**Purpose:** Player character and owned assets

**Description:** Contains player data including owned vehicles, trailers, assigned vehicles and various game settings.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `my_vehicles: N` | int | Number of owned vehicles |
| `my_vehicles[i]` | owner_ptr | References to player_vehicles units (with respect to modes)|
| `assigned_vehicles` | owner_ptr | Currently assigned vehicle |
| `trailers: N` | int | Number of trailers |
| `trailers[i]` | owner_ptr | References to trailer units |
| `trailer_utilization_logs[i]` | owner_ptr | References to trailer job logs |
| `trailer_defs[i]` | owner_ptr | References to trailer definitions |
| `current_job` | owner_ptr | References to current job details |
| `selected_job` | owner_ptr | References to selected job details |
| `trucks[i]` | owner_ptr | References to trucks units |
| `truck_profit_logs[i]` | owner_ptr | References to truck job logs |
| `drivers[i]` | link_ptr | References to driver details |

**Example:**
```sii
player : _nameless.254.0377.2108 {
 hq_city: calais
 my_vehicles_mode: truck
 my_vehicles: 3
 my_vehicles[0]: _nameless.253.ee87.5a48
 my_vehicles[1]: _nameless.253.ee87.6088
 my_vehicles[2]: _nameless.253.ee87.6308
 assigned_vehicles_mode: truck
 assigned_vehicles: _nameless.253.ee87.5728
 trailers: 17
 trailers[0]: _nameless.254.0c8e.4918
 trailers[1]: _nameless.254.0c8e.35b8
 ...
 trailers[16]: _nameless.254.0c8e.71c8
 trailer_utilization_logs: 17
 trailer_utilization_logs[0]: _nameless.254.0cb6.faa8
 trailer_utilization_logs[1]: _nameless.254.0cb6.f208
 ...
 trailer_utilization_logs[16]: _nameless.254.0cb7.0fa8
 trailer_defs: 13
 trailer_defs[0]: _nameless.250.e46a.4308
 trailer_defs[1]: _nameless.250.e46a.28c8
 ...
 trailer_defs[12]: _nameless.250.e46a.37c8
 buses: 0
 cars: 0
 schedule_transfer_to_hq: false
 schedule_quick_travel: false
 flags: 25116653
 vehicle_mode_flags: 0
 gas_pump_money_debt: 0
 current_job: null
 current_bus_job: null
 current_car_job: null
 selected_job: null
 selected_bus_offer: null
 driving_time: 121
 last_sleep_time: 0
 time_since_mandatory_break: 0
 on_break_time: 0
 free_roam_distance: 45
 discovary_distance: &4683d88d                  # Probably a typo by SCS
 dismissed_drivers: 0
 trucks: 8
 trucks[0]: _nameless.253.dd65.c488
 trucks[1]: _nameless.253.dd65.c6b8
 ...
 trucks[7]: _nameless.253.dd65.a818
 truck_profit_logs: 8
 truck_profit_logs[0]: _nameless.253.e9b8.1f68
 truck_profit_logs[1]: _nameless.253.e9b8.1c58
 ...
 truck_profit_logs[7]: _nameless.253.e9b8.2908
 drivers: 1
 drivers[0]: driver.0
 driver_flags: 1
 driver_flags[0]: 0
 driver_readiness_timer: 1
 driver_readiness_timer[0]: 0
 driver_undrivable_truck_timers: 1
 driver_undrivable_truck_timers[0]: 0
 driver_quit_warned: 0
 assigned_truck: null
 truck_placement: (&4775127e, &4184ad9c, &46f34798) (&bc2b81c1; &b9249bdb, &bf7ffc6a, &b96cf0af)
 assigned_trailer: null
 trailer_placement: (0, 0, 0) (1; 0, 0, 0)
 slave_trailer_placements: 0
 assigned_trailer_connected: false
 my_truck: null
 my_truck_placement: (&4728743c, &4159503f, &c75d73c7) (&3f5d05b5; &b784367e, &3f012bc4, &b9d01c16)
 my_truck_placement_valid: false
 my_trailer: null
 my_trailer_placement: (0, 0, 0) (1; 0, 0, 0)
 my_slave_trailer_placements: 0
 my_trailer_attached: false
 my_trailer_used: false
}
```

**Reference Chain to This Unit:**
```
economy → player
```

**Regex to locate:**
```regex
player\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## player_vehicles

**Purpose:** Container for player vehicle configurations

**Description:** Each player_vehicles unit contains references to actual vehicle/trailer units and their storage locations.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `vehicle` | owner_ptr | Reference to vehicle unit (if assigned) |
| `trailer` | owner_ptr | Reference to trailer unit (if attached) |
| `stored_vehicle_placement` | placement | Position/rotation when stored |
| `stored_vehicle_city` | string | City name when stored |
| `stored_vehicle_country` | string | Country when stored |

**Example:**
```sii
player_vehicles : _nameless.253.ee87.5a48 {
 vehicle: _nameless.253.dd65.c488
 stored_vehicle_placement: (&c6ea5dee, &4134ddfa, &c5930b18) (&39b702d4; &32494d54, 1, &37a8d636)
 stored_vehicle_city: null
 stored_vehicle_country: null
 trailer: null
 stored_trailer_placements: 0
 stored_trailer_attached: false
}
```

**Reference Chain to This Unit:**
```
economy → player → my_vehicles[i]
economy → player → assigned_vehicles
```

**Regex to locate:**
```regex
player_vehicles\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## trailer

**Purpose:** Actual trailer units with their current state

**Description:** Contains trailer-specific data including cargo, wear, accessories, and configuration references.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `trailer_definition` | owner_ptr | Reference to trailer_def unit |
| `cargo_mass` | int | Current cargo weight |
| `cargo_damage` | float | Cargo damage (0.0 = none) |
| `trailer_body_wear` | float | Average damage of part (hex float) |
| `trailer_body_wear_unfixable` | float | Average wear of part (hex float) |
| `accessories: N` | int | Number of accessories |
| `accessories[i]` | owner_ptr | References to accessory units |
| `odometer` | int | Total distance |
| `integrity_odometer` | int | Used to calculate maintenance % of trailer |
| `license_plate` | string | License plate with formatting |


**Example:**
```sii
trailer : _nameless.254.0c8e.4918 {
 trailer_definition: _nameless.250.e46a.4308
 oversize: false
 cargo_mass: 0
 cargo_damage: 0
 virtual_rear_wheels_offset: 0
 slave_trailer: null
 is_private: false
 trailer_body_wear: &372a902e
 trailer_body_wear_unfixable: &35087358
 accessories: 20
 accessories[0]: _nameless.254.0cb5.ab08
 accessories[1]: _nameless.254.0cae.4a48
 ...
 accessories[19]: _nameless.254.0cae.4f88
 odometer: 3
 odometer_float_part: &3e63db02
 integrity_odometer: 3
 integrity_odometer_float_part: &3e63db02
 trip_fuel_l: 0
 trip_fuel: 0
 trip_recuperation_kwh: 0
 trip_recuperation: 0
 trip_distance_km: 0
 trip_distance: 0
 trip_time_min: 0
 trip_time: 0
 license_plate: "DWR 58PF|poland"
 license_plate_type: ""
 chassis_wear: &3755343a
 chassis_wear_unfixable: &352a902e
 wheels_wear: 3
 wheels_wear[0]: &3704a4e4
 wheels_wear[1]: &3704a4e4
 wheels_wear[2]: &3704a4e4
 wheels_wear_unfixable: 3
 wheels_wear_unfixable[0]: &387c3116
 wheels_wear_unfixable[1]: &387c3116
 wheels_wear_unfixable[2]: &387c3116
 sliding_axle_offset: 0
}
```

**Reference Chain to This Unit:**
```
economy → player → my_vehicles[i] → player_vehicles → trailer
economy → player → assigned_vehicles → player_vehicles → trailer
economy → player → trailers[i] → trailer
```

**Regex to locate:**
```regex
trailer\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Repair trailer:** Change all `*_wear` fields to `&00000000` or `0`
- **Clear cargo damage:** Change `cargo_damage` to `&00000000` or `0`
- **Reset mileage:** Change `odometer` to `0`

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## trailer_utilization_log

**Purpose:** Trailer usage history

**Description:** Tracks trailer usage including distance and cargo transported.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `entries: N` | int | Number of log entries |
| `entries[i]` | owner_ptr | References to trailer_utilization_log_entry units |
| `total_driven_distance_km` | int | Total distance |
| `total_transported_cargoes` | int | Total cargo units |
| `total_transported_weight` | float | Total weight |

**Example:**
```sii
trailer_utilization_log : _nameless.254.0cb7.0768 {
 entries: 2
 entries[0]: _nameless.253.d5ed.cdf8
 entries[1]: _nameless.253.d5ed.d3c8
 total_driven_distance_km: 940
 total_transported_cargoes: 1
 total_transported_weight: &46fff4cd
}
```

**Reference Chain to This Unit:**
```
economy → player → trailer_utilization_logs[i]
```

**Regex to locate:**
```regex
trailer_utilization_log\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## trailer_utilization_log_entry

**Purpose:** Individual trailer usage log entry

**Description:** Records a single period of trailer usage.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `economy_day` | int | Day of economy time |
| `use_time` | int | Duration in minutes |

**Example:**
```sii
trailer_utilization_log_entry : _nameless.253.d5ed.cdf8 {
 economy_day: 23
 use_time: 929
}
```

**Reference Chain to This Unit:**
```
economy → player → trailer_utilization_logs[i] → trailer_utilization_log → entries[i]
```

**Regex to locate:**
```regex
trailer_utilization_log_entry\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## trailer_def

**Purpose:** Trailer blueprint/definition templates

**Description:** Defines trailer specifications that are shared by multiple trailer units. Used by the game to generate trailers for jobs.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `trailer` | string | Reference to trailer unit name |
| `gross_trailer_weight_limit` | float | Max weight (kg) |
| `chassis_mass` | float | Chassis weight (kg) |
| `body_mass` | int/float | Body weight (kg) |
| `axles` | int | Number of axles |
| `volume` | float | Cargo volume (m³) |
| `body_type` | token | `flatbed`, `refrigerated`, `curtain`, etc. |
| `chain_type` | token | `single`, `hct` |
| `country_validity: N` | int | Number of countries that is valid |
| `country_validity[i]` | string | Actual country name |
| `mass_ratio[i]` | int/float | Ratio of mass of each trailer section |
| `length` | float | Trailer length |
| `source_name` | string | Definition file name |

**Example:**
```sii
trailer_def : _nameless.1e9.6c09.e668 {
 trailer: ""
 gross_trailer_weight_limit: 84000
 chassis_mass: 12700
 body_mass: 400
 axles: 8
 volume: 160
 body_type: container
 chain_type: hct
 country_validity: 2
 country_validity[0]: finland
 country_validity[1]: sweden
 mass_ratio: 3
 mass_ratio[0]: &3ee66666
 mass_ratio[1]: &3dcccccd
 mass_ratio[2]: &3ee66666
 length: &41e4cccd
 source_name: "trailer_def.scs.flatbed.hct_3_2_3.container"
}
```

**Reference Chain to This Unit:**
```
economy → player → trailer_defs[i]
trailer → trailer_definition
```

**Regex to locate:**
```regex
trailer_def\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## selected/current job

**Purpose:** Contains information about the currently selected job and the active job being played

**Description:** The game maintains two job references - `selected_job` (the job that is highlighted/selected in the job board) and `current_job` (the job actively being played). When no job is selected, these are set to `null`. When a job is accepted, it becomes the `current_job`.

**Key Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `cargo` | link_ptr | Reference to cargo definition |
| `source_company` | link_ptr | Reference to source company |
| `target_company` | link_ptr | Reference to target company |
| `cargo_model_index` | int | Index of cargo model |
| `is_articulated` | bool | Whether the cargo requires articulated connection |
| `is_cargo_market_job` | bool | Whether the job was from cargo market |
| `start_time` | int | Game time when job started |
| `planned_distance_km` | int | Planned distance in kilometers |
| `ferry_time` | int | Ferry travel time in minutes |
| `ferry_price` | int | Ferry cost |
| `urgency` | int | Urgency level (0-2) |
| `special` | null | Special job flags |
| `car` | null | Car transport (null if none) |
| `units_count` | int | Number of cargo units |
| `fill_ratio` | int | Tank fill ratio (1 = 100%) |
| `company_truck` | owner_ptr | Company truck reference (null if player's truck) |
| `company_trailer` | owner_ptr | Company trailer reference (null if player's trailer) |
| `target_placement` | placement | Target placement position |
| `target_placement_medium` | placement | Medium target placement position |
| `target_placement_hard` | placement | Hard target placement position |
| `target_placement_rigid` | placement | Rigid target placement position |
| `source_placement` | placement | Source placement position |
| `selected_target` | int | Selected target indicator |
| `time_lower_limit` | int | Lower time limit for delivery |
| `time_upper_limit` | int | Upper time limit for delivery |
| `job_distance` | int | Distance traveled on this job |
| `fuel_consumed` | int | Fuel consumed on this job |
| `last_reported_fuel` | float | Last reported fuel level (hex float) |
| `total_fines` | int | Total fines incurred |
| `is_trailer_loaded` | bool | Whether trailer is loaded |
| `online_job_id` | nil | World of Trucks job ID |
| `online_job_trailer_model` | null | World of Trucks job trailer model |
| `autoload_used` | bool | Whether autoload was used |

### selected_job

**Purpose:** Contains details about the currently selected job in the job board

**Example:**
```sii
job_info : _nameless.1d5.a383.d058 {
 cargo: cargo.furniture
 source_company: company.volatile.stokes.calais
 target_company: company.volatile.ika_bohag.rennes
 cargo_model_index: 38281
 is_articulated: false
 is_cargo_market_job: false
 start_time: 0
 planned_distance_km: 618
 ferry_time: 0
 ferry_price: 0
 urgency: 1
 special: null
 car: null
 units_count: 35
 fill_ratio: 1
}
```

**Reference Chain:**
```
economy → player → selected_job
```

### current_job

**Purpose:** Contains details about the currently active job being played

**Example:**
```sii
player_job : _nameless.1d4.e273.5148 {
 company_truck: null
 company_trailer: _nameless.1d4.70a2.1218
 target_placement: (&c73c96cf, &42100c00, &45c9cad8) (&3e85dcce; 0, &3f771877, 0)
 target_placement_medium: (&c73cc799, &42100c00, &45ca79e0) (&befeca70; 0, &3f5e0cec, 0)
 target_placement_hard: (0, &60ad78ec, 0) (1; 0, 0, 0)
 target_placement_rigid: (0, &60ad78ec, 0) (1; 0, 0, 0)
 source_placement: (0, 0, 0) (1; 0, 0, 0)
 selected_target: 1
 time_lower_limit: 38303
 time_upper_limit: 38553
 job_distance: 0
 fuel_consumed: 0
 last_reported_fuel: &43b7b3f0
 total_fines: 0
 is_trailer_loaded: true
 online_job_id: nil
 online_job_trailer_model: null
 autoload_used: true
 cargo: cargo.furniture
 source_company: company.volatile.stokes.calais
 target_company: company.volatile.ika_bohag.rennes
 cargo_model_index: 38281
 is_articulated: false
 is_cargo_market_job: false
 start_time: 37227
 planned_distance_km: 618
 ferry_time: 0
 ferry_price: 0
 urgency: 1
 special: null
 car: null
 units_count: 35
 fill_ratio: 1
}
```

**Reference Chain:**
```
economy → player → current_job
```

**Regex to locate:**
```regex
(job_info|player_job)\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Cancel job:** Change `current_job` or `selected_job` in player unit to `null`
- **Adjust urgency:** Change `urgency` to `0` (no urgency) or `2` (maximum urgency)
- **Increase time limit:** Change `time_upper_limit` to extend deadline
- **Create jobs:** Create jobs using this format

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## vehicle

**Purpose:** Actual truck units with their current state

**Description:** Contains truck-specific data including wear, fuel, accessories, and configuration.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `engine_wear` | float | Average damage of part (hex float) |
| `engine_wear_unfixable` | float | Average wear of part (hex float) |
| `fuel_relative` | float | Fuel level (0.0 = empty, 1.0 = full, can go beyond 1.0) |
| `user_mirror_rot: N` | int | Number of mirror settings |
| `user_mirror_rot[i]` | quaternion | Mirror rotation angles |
| `user_head_offset` | float3 | Driver head position |
| `user_wheel_up_down`, `user_wheel_front_back` | int | Wheel positions |
| `accessories: N` | int | Number of accessories |
| `accessories[i]` | owner_ptr | References to accessory units |
| `odometer` | int | Total distance |
| `integrity_odometer` | int | Used to calculate maintenance % of truck |
| `license_plate` | string | License plate with formatting |

**Example:**
```sii
vehicle : _nameless.253.dd65.c488 {
 engine_wear: &3aa086e5
 transmission_wear: &3c304cd4
 cabin_wear: &3d7c5ede
 engine_wear_unfixable: &3ba75df8
 transmission_wear_unfixable: &3bad07ee
 cabin_wear_unfixable: &3d4d97f7
 fuel_relative: &3f1d58b9
 rheostat_factor: &3f400000
 user_mirror_rot: 9
 user_mirror_rot[0]: (1; 0, 0, 0)
 ...
 user_mirror_rot[8]: (1; 0, 0, 0)
 user_head_offset: (0, 0, 0)
 user_fov: 0
 user_wheel_up_down: 0
 user_wheel_front_back: 0
 user_mouse_left_right_default: 0
 user_mouse_up_down_default: 0
 accessories: 60
 accessories[0]: _nameless.254.0cb7.1ea8
 accessories[1]: _nameless.253.e9b5.7b28
 ...
 accessories[59]: _nameless.254.0cae.c488
 odometer: 5824
 odometer_float_part: &3ee5b9f0
 integrity_odometer: 5824
 integrity_odometer_float_part: &3ee5b9f0
 license_plate: "BJ-965-NQ.<offset hshift=5 vshift=-5><img src=/material/ui/lp/france/62.mat color=FFFFFFFF width=16 height=34>|france"
 chassis_wear: &3d9dbb4b
 chassis_wear_unfixable: &3d26f84b
 wheels_wear: 3
 wheels_wear[0]: &3c51580d
 wheels_wear[1]: &3c514f9e
 wheels_wear[2]: &3c51580d
 wheels_wear_unfixable: 3
 wheels_wear_unfixable[0]: &3d79513f
 wheels_wear_unfixable[1]: &3d755d57
 wheels_wear_unfixable[2]: &3d79513f
 sliding_axle_offset: 0
}
```

**Reference Chain:**
```
economy → player → my_vehicles[i] → player_vehicles → vehicle
economy → player → assigned_vehicles → player_vehicles → vehicle
economy → player → trucks[i] → vehicle
```

**Regex to locate:**
```regex
vehicle\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Repair truck:** Change all `*_wear` fields to `&00000000` or `0`
- **Full fuel:** Change `fuel_relative` to `&3f800000` or `1` or more
- **Reset mileage:** Change `odometer` to `0`
- **Reset maintenance %:** Change `integrity_odometer` to `0`
- **Change plate:** Modify the `license_plate` string

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## vehicle accessories

**Purpose:** Comprehensive vehicle parts and customization components

**Description:** This category encompasses all vehicle accessories including basic parts (chassis, engine, cabin), wheels, addons (mirrors, badges, lights), and paint jobs. Each accessory type references SII data files that define specific component properties.

**Key Fields by Subtype:**

| Field | Type | Description | Applicable Subtypes |
|-------|------|-------------|---------------------|
| `data_path` | string | Path to SII file with accessory data | All |
| `refund` | int | Refund amount | All |
| `paint_color` | float3 | RGB color values | Wheels, Addons |
| `offset` | uint | Axle index (0 = front, 2+ = rear) | Wheels |
| `slot_name: N` | int | Number of slots | Addons |
| `slot_name[i]` | token | Slot locator name (e.g., `slot_0`) | Addons |
| `slot_hookup: N` | int | Number of hookups | Addons |
| `slot_hookup[i]` | string | Hookup unit name (e.g., `light2.addon_hookup`) | Addons |
| `base_color` | float3 | Base RGB color | Paint Jobs |
| `mask_r_color`, `mask_g_color`, `mask_b_color` | float3 | Mask colors | Paint Jobs |
| `flake_color` | float3 | Flakes color | Paint Jobs |
| `flip_color` | float3 | Flip color | Paint Jobs |

### vehicle_accessory (Basic Parts)

**Purpose:** Basic truck/trailer parts (chassis, engine, cabin, etc.)

**Example:**
```sii
vehicle_accessory : _nameless.254.0cb7.1ea8 {
 data_path: "/def/vehicle/truck/scania.r_2016/data.sii"
 refund: 0
}
```

### vehicle_addon_accessory (Addons)

**Purpose:** Added accessories (mirrors, badges, lights, etc.)

**Example:**
```sii
vehicle_addon_accessory : _nameless.254.0cae.9cc8 {
 slot_name: 0
 slot_hookup: 0
 paint_color: (1, 1, 1)
 data_path: "/def/vehicle/truck/scania.r_2016/accessory/doorhndl/stock_blk.sii"
 refund: 480
}

vehicle_addon_accessory : _nameless.254.0cae.9d88 {
 slot_name: 8
 slot_name[0]: slot_9
 slot_name[1]: slot_4
 slot_name[2]: slot_2
 slot_name[3]: slot_7
 slot_name[4]: slot_6
 slot_name[5]: slot_5
 slot_name[6]: slot_3
 slot_name[7]: slot_8
 slot_hookup: 8
 slot_hookup[0]: "strobe01.addon_hookup"
 slot_hookup[1]: "strobe01.addon_hookup"
 slot_hookup[2]: "strobe01.addon_hookup"
 slot_hookup[3]: "strobe01.addon_hookup"
 slot_hookup[4]: "strobe01.addon_hookup"
 slot_hookup[5]: "strobe01.addon_hookup"
 slot_hookup[6]: "strobe01.addon_hookup"
 slot_hookup[7]: "strobe01.addon_hookup"
 paint_color: (1, 1, 1)
 data_path: "/def/vehicle/truck/scania.r_2016/accessory/f_inlay_cab/stock.sii"
 refund: 1400
}
```

### vehicle_paint_job_accessory (Paint Jobs)

**Purpose:** Paint job configuration

**Example:**
```sii
vehicle_paint_job_accessory : _nameless.253.dacc.f6f8 {
 mask_r_color: (&3e080ea3, &3e10a754, &3e32d6c6)
 mask_g_color: (&3f6571ee, &3eaf0f6c, 0)
 mask_b_color: (0, 0, 0)
 flake_color: (0, 1, 0)
 flip_color: (1, 0, 0)
 base_color: (&3c0c6152, &3c6ca5df, &3d0798df)
 data_path: "/def/vehicle/truck/scania.r_2016/paint_job/tr_hd4_s_r_a.sii"
 refund: 20000
}
```

### vehicle_wheel_accessory (Wheels)

**Purpose:** Wheels, hubs, nuts, and other wheel-related accessories

**Example:**
```sii
vehicle_wheel_accessory : _nameless.253.e9b5.7b28 {
 offset: 0
 paint_color: (1, 1, 1)
 data_path: "/def/vehicle/f_disc/front_disc_02_matt_gray.sii"
 refund: 1200
}
```

**Reference Chain to This Unit:**
```
vehicle → accessories[i]
trailer → accessories[i]
```

**Regex to locate all types:**
```regex
vehicle(_wheel|_addon|_paint_job)?_accessory\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## truck_profit_logs

**Purpose:** Truck profit/expense history

**Description:** Contains aggregated profit statistics with detailed entry references.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `stats_data: N` | int | Number of entries |
| `stats_data[i]` | owner_ptr | References to profit_log_entry units |
| `acc_distance_free` | int | Accumulated non-job distance |
| `acc_distance_on_job` | int | Accumulated job distance |
| `history_age` | nil/int | History age indicator |

**Example:**
```sii
profit_log : _nameless.253.e9b8.1f68 {
 stats_data: 31
 stats_data[0]: _nameless.254.0317.bae8
 stats_data[1]: _nameless.254.0317.b308
 ...
 stats_data[30]: _nameless.254.0317.e368
 acc_distance_free: 500
 acc_distance_on_job: 1474
 history_age: nil
}
```

**Reference Chain to This Unit:**
```
economy → player → truck_profit_logs[i]
```

**Regex to locate:**
```regex
profit_log\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## profit_log_entry

**Purpose:** Individual profit/expense entry

**Description:** Records a single income or expense transaction.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `revenue` | int | Money earned |
| `wage` | int | Driver wage |
| `maintenance` | int | Repair costs |
| `fuel` | int | Fuel costs |
| `distance` | int | Distance traveled |
| `distance_on_job` | bool | Whether distance was on a job |
| `cargo_count` | int | Number of cargo units |
| `cargo` | string | Cargo type |
| `source_city`, `destination_city` | string | Job cities |
| `timestamp_day` | int | Day of transaction |

**Example:**
```sii
profit_log_entry : _nameless.254.0317.bae8 {
 revenue: 0
 wage: 0
 maintenance: 9291
 fuel: 0
 distance: 0
 distance_on_job: false
 cargo_count: 0
 cargo: ""
 source_city: ""
 source_company: ""
 destination_city: ""
 destination_company: ""
 timestamp_day: 21
}
```

**Reference Chain to This Unit:**
```
profit_log → stats_data[i]
```

**Regex to locate:**
```regex
profit_log_entry\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## company

**Purpose:** Company data with job offers

**Description:** Contains company-specific data including job offers and cargo information.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `permanent_data` | link_ptr | Reference to company_permanent unit |
| `job_offer: N` | int | Number of active job offers |
| `job_offer[i]` | owner_ptr | References to job_offer_data units |
| `cargo_offer_seeds: N` | int | Number of cargo seeds |
| `cargo_offer_seeds[i]` | int | Cargo seed values |

**Example:**
```sii
company : company.volatile.bhv.kristiansand {
 permanent_data: company.permanent.bhv
 delivered_trailer: null
 job_offer: 1
 job_offer[0]: _nameless.254.2ac7.8018
 cargo_offer_seeds: 10
 cargo_offer_seeds[0]: 38141
 cargo_offer_seeds[1]: 38682
 ...
 cargo_offer_seeds[9]: 38342
 discovered: false
 reserved_trailer_slot: nil
 state: 0
}
```

**Reference Chain to This Unit:**
```
economy → companies[i]
```

**Regex to locate:**
```regex
company\s*:\s*(company\.(permanent|volatile)\.\S+)\s*\{
```

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## job_offer_data

**Purpose:** Available job offers

**Description:** Contains job details including cargo, origin, destination, and pricing.

**Key Fields:**
| Field | Type | Description |
|-------|------|-------------|
| `target` | string | Destination company ID |
| `expiration_time` | int | Time when offer expires |
| `urgency` | int | Urgency level (0-3) |
| `shortest_distance_km` | int | Distance in kilometers |
| `ferry_time` | int | Ferry travel time in minutes |
| `ferry_price` | int | Ferry cost |
| `cargo` | link_ptr | Reference to cargo unit |
| `company_truck` | string | Provided truck type |
| `trailer_variant` | string | Offered trailer variant |
| `trailer_definition` | string | Provided trailer definition |
| `units_count` | int | Number of cargo units |
| `fill_ratio` | int | Truck tank fill ratio (1 = 100%) |

**Example:**
```sii
job_offer_data : _nameless.254.2ac7.8018 {
 target: "rosmark_ru.kaliningrad"
 expiration_time: 38346
 urgency: 2
 shortest_distance_km: 1419
 ferry_time: 135
 ferry_price: 468
 cargo: cargo.motorcycles
 company_truck: daf_xg_4x2_450
 trailer_variant: trailer.scs_dry_m
 trailer_definition: trailer_def.scs.box.single_3.dryvan_m
 units_count: 33
 fill_ratio: 1
 trailer_place: 0
}
```

**Reference Chain to This Unit:**
```
company → job_offer[i]
```

**Regex to locate:**
```regex
job_offer_data\s*:\s*_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\s*\{
```

**Common Edits:**
- **Remove urgency:** Change `urgency` to `0`
- **Extend deadline:** Change `expiration_time` to a larger number
- **Create jobs:** Create jobs using this format
---

<br>

[⬆ Back to Table of Contents](#table-of-contents)

## And More...

The save file contains many more unit types for various game systems:

| Unit Type | Purpose |
|-----------|---------|
| `economy_event` | Time-triggered economy events |
| `delivery_log_entry` | Delivery records |
| `ferry_log_entry` | Ferry trip records |
| `oversize_offer` | Oversize cargo job offers |
| `garage` | Garage facilities with vehicles/drivers |

---

## Working with Arrays


## Fixed Arrays (With Size Declaration)
```sii
accessories: 20
accessories[0]: _nameless.254.0cb5.ab08
accessories[1]: _nameless.254.0cae.4a48
accessories[2]: _nameless.254.0cae.2a08
...
accessories[19]: _nameless.254.0cae.4f88
```

**To add an item:**
1. Increment the count
2. Add a new `field[N]: value` line

**To remove an item:**
1. Decrement the count
2. Delete the corresponding `field[N]: value` line

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## Common Edits Reference

| What to Edit | Field Path | Example Value |
|--------------|------------|---------------|
| **Full fuel** | `vehicle : XXX { fuel_relative` | `&3f800000` |
| **Zero wear** | `vehicle : XXX { *_wear` | `&00000000` |
| **Reset mileage** | `vehicle : XXX { odometer` | `0` |
| **Max money** | `bank : XXX { money_account` | `999999999999` |
| **Max XP** | `economy : XXX { experience_points` | `9999999` |
| **Unlock all ADR skills** | `economy : XXX { adr` | `63` |
| **Unlock all driving skills** | `economy : XXX { long_dist` etc. | `6` |
| **Repair truck** | All `*_wear` fields | `&00000000` |
| **Repair trailer** | All `*_wear` fields | `&00000000` |

<br>

[⬆ Back to Table of Contents](#table-of-contents)

---

## Regex Reference

## Essential Patterns

| Pattern | Regex | Use Case |
|---------|-------|----------|
| Hex floats | `&[0-9a-fA-F]{8}` | Finding float values |
| Nameless refs | `_nameless\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}\.[0-9a-f]{3,4}` | Finding references |
| Company IDs | `company\.(permanent\|volatile)\.\S+` | Finding companies |

## Unit Type Patterns

| Pattern | Regex | Use Case |
|---------|-------|----------|
| Economy | `^economy\s*:\s*_nameless\.` | Find economy unit |
| Bank | `^bank\s*:\s*_nameless\.` | Find bank units |
| Player | `^player\s*:\s*_nameless\.` | Find player units |
| Vehicle | `^vehicle\s*:\s*_nameless\.` | Find vehicle units |
| Trailer | `^trailer\s*:\s*_nameless\.` | Find trailer units |
| Company | `^company\s*:\s*(company\.)` | Find company units |

<br>

[⬆ Back to Table of Contents](#table-of-contents)

## License

This guide is licensed under the Creative Commons Attribution 4.0 International License (CC BY 4.0).  
You are free to share and adapt this material with attribution.

Code examples in this guide are provided under the MIT License.

## Disclaimer

This is an unofficial community guide. It is not affiliated with, endorsed by, or sponsored by SCS Software.  
Euro Truck Simulator 2, American Truck Simulator, and SCS Software are trademarks of SCS Software.  
Always back up your save files before editing.