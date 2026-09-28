# Home Assistant blueprints

Automation blueprints for [Home Assistant](https://www.home-assistant.io/). Import one with a single click and set it up in the automation editor, where every setting is explained. No YAML needed.

| Blueprint | What it does |
|---|---|
| [Appliance cycle counter](#appliance-cycle-counter) | Counts the runs of a washing machine, dishwasher, dryer or vacuum and reminds you when it needs maintenance. |
| [Button cycle brightness](#button-cycle-brightness) | Steps the brightness of lights up and down with a single button. |
| [Car mileage notification](#car-mileage-notification) | Warns you before your car passes the yearly km limit of its insurance or lease. |
| [Motion-triggered adaptive light](#motion-triggered-adaptive-light) | Turns a light on with motion when the room is dark, with a night mode. |
| [Temperature-controlled switch](#temperature-controlled-switch) | Turns a heater, fan or other switch on and off to keep a temperature between two limits. |

All blueprints need Home Assistant 2025.10 or newer.

## Install

Select the **Import blueprint** button of a blueprint below, then create an automation from it under **Settings > Automations & scenes > Blueprints**. To get a newer version later, pick **Re-import blueprint** in the blueprint's menu there. Your automations keep their settings.

## Appliance cycle counter

[![Import the Appliance cycle counter blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fappliance_cycle_counter_notification.yaml)

Counts finished cycles of an appliance and sends a notification to your phone when a limit is reached, for example to clean the washing machine every 30 washes.

- For appliances without their own counter, create a counter helper. The blueprint adds 1 each time the appliance's state changes, for example from `running` to `off`.
- Appliances with their own cycle sensor need no helper.
- It can remind you again every few cycles, and reset the counter after the notification.

## Button cycle brightness

[![Import the Button cycle brightness blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fbutton_cycle_brightness.yaml)

Hold the button to step the brightness up to 100%, then back down, then off. A short tap turns the lights off, or on. Works with Hue, Matter and Zigbee2MQTT buttons that report an `event_type`.

- A toggle helper remembers the direction. Without it, the cycle only goes up and turns off after 100%.
- You can set the step size and color temperature, keep the lights between off and 100% ("Limit extremes"), and turn them on at their last brightness.
- On a Hue wall switch module, set the switch type to **Push button** in the Hue app, or long presses do not work.

## Car mileage notification

[![Import the Car mileage notification blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fcar_mileage_notification.yaml)

Warns you on your phone when your car gets close to the yearly km limit of its insurance or lease, and again every few hundred km after that.

- Needs a sensor with the km driven this insurance year, for example a utility meter helper on the car's odometer with a yearly reset.
- Frequent odometer updates while driving are skipped, but a passed milestone is never missed.

## Motion-triggered adaptive light

[![Import the Motion-triggered adaptive light blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fmotion_triggered_adaptive_light.yaml)

Turns a light on with motion when the room is dark enough, and off again when the motion stops.

- Needs a motion sensor, a light sensor that measures lux, and a light with brightness and color temperature.
- Night mode uses a lower brightness and a warmer color, on a schedule or with a night sensor.
- Blocking entities, like a sleep mode helper, keep the light off. Other lights, switches or areas can follow along.
- After a Home Assistant restart the light turns off, so it does not stay on by mistake.

## Temperature-controlled switch

[![Import the Temperature-controlled switch blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Ftemperature_controlled_switch.yaml)

Turns a heater, fan or other switch on and off to keep a temperature between a low and a high limit. Heating mode turns it on when it is cold, cooling mode when it is hot.

- Checks every 5 minutes by default. A longer interval stops compressors from switching too often.
- Blocking entities, like an away mode helper, pause it. Other devices can follow along.

## License

[MIT](LICENSE)
