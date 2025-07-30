# Airspeed Validation

PX4 includes a set of airspeed validation checks that run continuously during fixed-wing flight to ensure the airspeed data is accurate and reliable. If any check fails persistently, the system can declare airspeed as invalid.

::: info
By default, the [Missing Data](#missing-data-check), [Data Stuck](#data-stuck-check), and [Innovation](#innovation-check) checks are enabled. You can configure which checks are active using the [ASPD_DO_CHECKS](../advanced_config/parameter_reference.md#ASPD_DO_CHECKS) parameter.
:::


## Overview of Validation Checks

This overview summarizes each check, common causes for failure, and the relevant parameters. To configure the delay before starting or stopping the use of airspeed sensor data after it passes or fails validation, see [ASPD_FS_T_START](../advanced_config/parameter_reference.md#ASPD_FS_T_START) and [ASPD_FS_T_STOP](../advanced_config/parameter_reference.md#ASPD_FS_T_STOP).

### Missing Data Check

::: info
This check is independent of the other validation checks and cannot be disabled or configured. It must pass for any of the other checks to run.
:::

- **Description**: Triggers when no new airspeed data has been received for more than 1 second.
- **Common Failure Causes**: Faulty or disconnected sensor.

### Data Stuck Check

- **Description** : Triggers when the measured indicated airspeed (IAS) is not changing for more than 2 seconds, indicating a potential blocked pitot tube.
- **Common Failure Causes** : Faulty or blocked airspeed sensor, very low sensor resolution.

### Innovation Check

- **Description** : Compares the estimated true airspeed (TAS) to the predicted GNSS-based airspeed (groundspeed minus windspeed). If the difference exceeds a threshold ([ASPD_FS_INNOV](../advanced_config/parameter_reference.md#ASPD_FS_INNOV)) for a prolonged period ([ASPD_FS_INTEG](../advanced_config/parameter_reference.md#ASPD_FS_INTEG)), the check fails.
- **Common Failure Causes** : Faulty or blocked airspeed sensor, poor GNSS data, inaccurate wind estimate.
- **Relevant Parameters** : [ASPD_FS_INNOV](../advanced_config/parameter_reference.md#ASPD_FS_INNOV), [ASPD_FS_INTEG](../advanced_config/parameter_reference.md#ASPD_FS_INTEG)

### Load Factor Check

- **Description** : Checks whether the measured airspeed is consistent with the aircraft load. It should be high enough to generate the required lift to sustain the current load on the vehicle.
- **Common Failure Causes** : Faulty or blocked airspeed sensor, incorrect stall speed, or the vehicle is approaching stall.
- **Relevant Parameters** : [FW_AIRSPD_STALL](../advanced_config/parameter_reference.md#FW_AIRSPD_STALL)

### First Principle Check

- **Description** : Checks whether the measured IAS is consistent with the aircraft throttle and pitch settings. Specifically, when the throttle is high (5% above trim) and the aircraft is pitched down (below FW_PSP_OFF): the IAS should be increasing accordingly.
- **Common Failure Causes** : Faulty or blocked airspeed sensor, pitot tube icing, excessive drag (e.g. flaps down, landing gear deployed, payload), throttle not producing expected thrust, incorrect trim or max throttle setting
- **Relevant Parameters** : [ASPD_FP_T_WINDOW](../advanced_config/parameter_reference.md#ASPD_FP_T_WINDOW), [FW_PSP_OFF](../advanced_config/parameter_reference.md#FW_PSP_OFF), [FW_THR_TRIM](../advanced_config/parameter_reference.md#FW_THR_TRIM), [FW_THR_MAX](../advanced_config/parameter_reference.md#FW_THR_MAX)


## TAS Scale Considerations

The True Airspeed (TAS) scale is a dynamic correction factor used to account for discrepancies between measured Indicated Airspeed (IAS) and the actual airspeed an aircraft experiences in flight. PX4 estimates this TAS scale using GNSS ground speed and wind estimation during fixed-wing flight.

This scaling plays an important role in keeping the [Innovation check](#innovation-check) reliable, since a well-estimated TAS is key to spotting inconsistencies between measured and predicted airspeed. If the TAS scale is incorrect, it can mask real airspeed faults or trigger false positives.

If the estimated scale appears consistently off, you can override it by setting a fixed value using [ASPD_SCALE_1](../advanced_config/parameter_reference.md#ASPD_SCALE_1), and disable the estimated scale with [ASPD_SCALE_APPLY](../advanced_config/parameter_reference.md#ASPD_SCALE_APPLY).
