---
title: "Ecobee"
description: "Instructions for how to integrate Ecobee thermostats and sensors within Home Assistant."
logo: ecobee.png
ha_category:
  - Sensor
  - Binary Sensor
  - Notifications
  - Climate
  - Weather
featured: true
ha_release: 0.9
ha_iot_class: Cloud Poll
redirect_from:
  - /components/sensor.ecobee/
  - /components/binary_sensor.ecobee/
  - /components/notify.ecobee/
  - /components/climate.ecobee/
  - /components/weather.ecobee/
---

The `ecobee` integration lets you control thermostats and view sensor data from [ecobee](https://ecobee.com) thermostats and remote sensors.

## Preliminary Step

You will need to obtain an API key from ecobee's [developer site](https://www.ecobee.com/developers/) to use this integration. To get the key, first, you need to register your thermostat which should be done as part of the ecobee installation. Once you have done that, perform the following steps.

1. Click on the **Become a developer** link on the [developer site](https://www.ecobee.com/developers/).
2. Log in with your ecobee credentials.
3. Accept the SDK agreement.
4. Fill in the fields.
5. Click **save**.

Now login to the regular consumer portal and in the hamburger menu, there will be a new option **Developer**. Now we can create an Application to link to Home Assistant.

1. Select the **Developer** option from the hamburger menu.
2. Select **Create New**.
3. Give your App a name (it must be unique across all ecobee users; try your-name-or-alias-home-assistant) and a summary (which need not be unique). Neither of these are important as they are not used anywhere in Home Assistant.
4. For authorization method select **ecobee PIN**.
5. You don't need an Application Icon or Detailed Description.
6. Click **Create**.

Now under the Name and Summary Section, you will have an API key. Copy this key as you will need it in the steps that follow. Click the **X** to close the Developer section.

## Configuring the Integration

To configure the ecobee integration in Home Assistant, you can either use the **Configuration** -> **Integrations** menu, or add an entry to `configuration.yaml`.

### Setup via the Integrations menu

1. In the **Configuration** -> **Integrations** menu, click **+** and then select `ecobee` from the pop-up menu.
2. In the pop-up box, enter the API key you obtained from ecobee.com.
3. In the next pop-up box, you will be presented with a unique four-character PIN code which you will need to authorize in the [ecobee consumer portal](https://www.ecobee.com/consumerportal/index.html). You can do this by logging in, selecting **My Apps** from the hamburger menu, clicking **Add Application** on the left, entering the PIN code from Home Assistant, and clicking **Validate** and then **Add Application** in the bottom right.
4. After authorizing the App on ecobee.com, return to Home Assistant and hit **Submit**. If the authorization was successful, a config entry will be created and your thermostats and sensors will be available in Home Assistant.

### Setup via configuration.yaml

If you prefer to initially set up this integration in [`configuration.yaml`](/docs/configuration/), you may do so by adding your API key (and optional parameters) as follows (however, you must still complete authorization via the **Integrations** menu):

```yaml
# Example configuration.yaml entry
ecobee:
  api_key: YOUR_API_KEY
```

[Restart Home Assistant](/docs/configuration/#reloading-changes) for the changes to take effect. In the **Configuration** -> **Integrations** menu, hit **Configure** next to the discovered `ecobee` entry, and continue to authorize the App per the Integration menu instructions above.

{% configuration %}
api_key:
  description: Your ecobee API key. This is only needed for the initial setup of the integration. Once registered it can be removed. If you revoke the key in the ecobee portal, you will need to remove the existing `ecobee` configuration in the **Integrations** menu, update this, and then configure the Integration again.
  required: false
  type: string
{% endconfiguration %}

<p class='img'>
  <img src='{{site_root}}/images/screenshots/ecobee-sensor-badges.png' />
  <img src='{{site_root}}/images/screenshots/ecobee-thermostat-card.png' />
</p>

## Notifications

To get your Ecobee notifications working with Home Assistant, you must first have the main Ecobee integration loaded and running. Once you have that configured, you can set up this integration to send messages to your Ecobee device.

To use this notification platform in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml entry
notify:
  - name: NOTIFIER_NAME
    platform: ecobee
```

{% configuration %}
name:
  description: Setting the optional parameter `name` allows multiple notifiers to be created. The notifier will bind to the service `notify.NOTIFIER_NAME`.
  required: false
  default: "`notify`"
  type: string
{% endconfiguration %}

To use notifications, please see the [getting started with automation page](/getting-started/automation/).

## Thermostat

### Concepts

The Ecobee Thermostat supports the following key concepts.

The _target temperature_ is the temperature that the device attempts
to achieve. The target temperature is either determined by the
currently active climate or it may be overridden by a hold. When the
thermostat is not in auto mode, there is a single target
temperature. When the thermostat is in auto HVAC mode, there is a
pair of target temperatures: the lower target temperature determines
the lowest desired temperature, while the higher target temperature
determines the highest desired temperature (the thermostat will switch
between heating and cooling to keep the temperature within these
limits).

A _climate_ is a predefined or user-defined set of presets that the
thermostat aims to achieve. The ecobee thermostat provides three predefined
climates: Home, Away, and Sleep. The user can define additional climates.

A _preset_ is an override of the target temperature defined in the
currently active climate. The temperature targeted in the preset mode may be
explicitly set (temperature preset), it may be derived from a reference
climate (home, away, sleep, etc.), or it may be derived from a vacation
defined by the thermostat. All holds are temporary. Temperature and
climate holds expire when the thermostat transitions to the next climate
defined in its program. A vacation hold starts at the beginning of the
defined vacation period, and expires when the vacation period ends.

When in _away preset_, the target temperature is permanently overridden by
the target temperature defined for the away climate. The away preset is a
simple way to emulate a vacation mode.

The _HVAC mode_ of the device is the currently active operational
modes that the Ecobee thermostat provides: heat, auxHeatOnly, cool,
auto, and off.

## Attributes

The Ecobee climate entity has some extra attributes to represent the state of the thermostat.

| Name | Description |
| ---- | ----------- |
| `fan` | If the fan is currently on or off: `on` / `off`.
| `climate_mode` | This is the climate mode that is active, or would be active if no override is active.
| `equipment_running` | This is a comma seperated list of equipment that is currently running.
| `fan_min_on_time` | The minimum amount of minutes that the fan will be on when it's turned on.

## Services

Besides the standard services provided by the Home Assistant [Climate](https://www.home-assistant.io/components/climate/) integration, the following extra services are provided by the Ecobee Thermostat: `ecobee.resume_program` and `ecobee.set_fan_min_on_time`.

### Service `ecobee.resume_program`

Resumes the currently active schedule.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of strings that point at `entity_id`'s of climate devices to control. Else targets all.
| `resume_all` | no | true or false

### Service `ecobee.set_fan_min_on_time`

Sets the minimum amount of time that the fan will run.

| Service data attribute | Optional | Description |
| ---------------------- | -------- | ----------- |
| `entity_id` | yes | String or list of strings that point at `entity_id`'s of climate devices to control. Else targets all.
| `fan_min_on_time` | no | integer (e.g. 5)
