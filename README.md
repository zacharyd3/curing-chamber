# curing-chamber

Home Assistant blueprint (`curing_chamber_controller.yaml`) that runs a charcuterie
curing chamber from one switched outlet (e.g. a Sonoff with power monitoring).

## Helpers to create

Required:

| Helper | Type | Settings |
| --- | --- | --- |
| Smoothed humidity | Statistics | Live humidity sensor, characteristic *mean*, max age 1 h |
| 24h average temperature / humidity | Statistics | mean, max age 24 h |

Optional:

| Helper | Type | Settings | Enables |
| --- | --- | --- | --- |
| Smoothed temperature | Statistics | Live temperature sensor, *mean*, max age 1 h | 1h temperature on the dashboard and in alerts, plus overshoot-tuning hints |
| 24h run time | History stats | Power switch, state `on`, type *ratio* (or *time*), end `{{ now() }}`, duration 24 h | Duty cycle in the daily summary |
| Memory | Text (`input_text`) | **Maximum length 255** | Cycle learning, the "Follow the learned cycle" failsafe, defrost |
| Status / detail text | Text (`input_text`) | default length | Dashboard readouts |

## Cycle learning and failsafe

With the memory helper set, every normal thermostat cycle (switched on and off by
the controller itself) is recorded as a running average of on-time and off-time
for each 3-hour slot of the day. Cycles touched by a manual toggle, the failsafe,
a defrost or a restart are skipped. A slot is used once it has 3 clean cycles of
each kind; slots still learning borrow from the nearest learned slot.

Set **If the temperature sensor goes unavailable or stale** to *Follow the learned
cycle* and the chamber keeps cycling on that schedule until the sensor returns.
If nothing has been learned yet, it turns power off.

A sensor counts as stale when it still shows a value but has not reported for the
**Stale sensor timeout** (default 60 min). Clear the memory helper to relearn from
scratch, for example after moving the chamber.
