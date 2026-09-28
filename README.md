# Home Assistant blueprints

Automation blueprints for [Home Assistant](https://www.home-assistant.io/). Import one with a single click and set it up in the automation editor, no YAML needed.

| Blueprint | What it does |
|---|---|
| [Appliance cycle counter](#appliance-cycle-counter) | Counts the runs of a washing machine, dishwasher, dryer or vacuum and reminds you on your phone when it needs maintenance. |
| [Button cycle brightness](#button-cycle-brightness) | Steps the brightness of one or more lights up and down with a single button. |
| [Motion-triggered adaptive light](#motion-triggered-adaptive-light) | Turns a light on with motion when the room is dark, with a dimmer and warmer night mode. |

All blueprints need Home Assistant 2025.10 or newer.

## Install a blueprint

1. Select the **Import blueprint** button of the blueprint below. Home Assistant opens and asks you to import it.
2. Go to **Settings > Automations & scenes > Blueprints**, select the blueprint and fill in the settings. Save, and the automation runs.

You can create as many automations from one blueprint as you like, for example one per room or one per appliance.

Without the button: go to **Settings > Automations & scenes > Blueprints**, select **Import blueprint** and paste the link to the blueprint file.

To get a newer version later, open **Settings > Automations & scenes > Blueprints**, select the three dots next to the blueprint and pick **Re-import blueprint**. Your automations keep their settings.

More about blueprints: [Using automation blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/).

## Appliance cycle counter

[![Import the Appliance cycle counter blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fmm98%2Fappliance_cycle_counter.yaml)

Counts finished cycles of an appliance and sends a notification to your phone when a limit is reached. Use it to remember maintenance: clean the washing machine every 30 washes, replace the vacuum filter after 50 runs or descale the coffee machine after 200 brews.

### Two ways to count

- **The appliance does not count by itself.** Create a counter helper: go to **Settings > Devices & services > Helpers**, select **Create helper** and pick **Counter**, with initial value 0, step 1 and minimum 0. Select it as the counter entity. Then select the appliance's state entity and the change that marks a finished cycle, for example from `running` to `off`. Each time the appliance makes that change, the blueprint adds 1 to the counter. A number helper (`input_number`) works too.
- **The appliance counts by itself.** Some vacuums and washing machines have a sensor with their number of cycles. Select that sensor as the counter entity and leave the state entity empty. The blueprint only watches the sensor.

### How it works

- A cycle only counts when the state changes from exactly the "from" state to exactly the "to" state. Other changes, like `idle` to `off`, do not count. The state names are case-sensitive. To see the exact names, open the entity's history, or look it up under **Developer tools > States**.
- When the counter reaches the limit, your phone gets a notification.
- With a re-notify interval, you get another notification every that many cycles after the limit, as long as the counter is not reset. With limit 30 and interval 5, that is at 30, 35, 40 and so on.
- With **Reset counter after notification** on, the counter goes back to 0 right after the notification, so the count starts again. The blueprint cannot reset a sensor that belongs to the appliance.
- Lowering the counter yourself, for example resetting it after cleaning the machine, never sends a notification.

### Example

A washing machine whose state entity goes from `running` to `off` when a wash is done:

- Counter entity: a counter helper named "Washing machine cycles"
- State entity: the washing machine's state entity, from `running` to `off`
- Upper limit: 30, and **Reset counter after notification** on
- Message: `The washing machine has run {{ count }} washes. Time for a cleaning cycle.`

### Settings

| Setting | Default | What it does |
|---|---|---|
| Counter entity | | The counter helper, number helper or the appliance's own cycle sensor. Required. |
| Notification device | | The phone or tablet with the Home Assistant Companion app that gets the notification. Required. |
| Notification title | `🔧 Maintenance Reminder` | The title of the notification. |
| Notification message | `Counter reached {{ count }} of {{ limit }} cycles. Maintenance is required.` | The text of the notification. `{{ count }}` is replaced by the current count and `{{ limit }}` by the upper limit. |

**State transition counting**

| Setting | Default | What it does |
|---|---|---|
| State entity | empty | The entity whose change marks a finished cycle, for example the washing machine's state. Leave it empty when the counter entity is the appliance's own sensor. |
| From state | `running` | The state before the change, for example `running`, `on` or `cleaning`. |
| To state | `off` | The state after the change, for example `off`, `idle`, `finished` or `docked`. |

**Limits and thresholds**

| Setting | Default | What it does |
|---|---|---|
| Upper limit | 30 | The count that sends the notification, from 1 to 10000. |
| Re-notify interval | 0 | Send another notification every this many cycles after the limit. 0 sends only one. |

**Reset**

| Setting | Default | What it does |
|---|---|---|
| Reset counter after notification | off | Sets a counter or number helper back to 0 after the notification. |

**Debug**

| Setting | Default | What it does |
|---|---|---|
| Log to activity | off | Writes every finished cycle and every counter change to the logbook, with the count, the limit and whether a notification was sent. |

## Button cycle brightness

[![Import the Button cycle brightness blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fmm98%2Fbutton_cycle_brightness.yaml)

Steps the brightness of one or more lights with a single button. Each press moves one step up until the lights are at full brightness, then one step down, then off, and around again. A short tap turns the lights off, or on when they are off.

It works with buttons that report their presses as an event entity with an `event_type`, like Hue, Matter and Zigbee2MQTT buttons, remotes, dimmer switches and wall switch modules.

### How the cycle works

- By default, holding the button (a long press) moves one step, and a short tap turns the lights off. When the lights are off, the tap turns them on.
- With a step of 20% the cycle is: 20%, 40%, 60%, 80%, 100%, 80%, 60%, 40%, 20%, off, 20% and so on.
- The brightness always lands on a multiple of the step. If something else set the light to 30%, the next press goes to 40% on the way up or 20% on the way down.
- When the lights are off, the first press turns them on at the step percentage. With **Restore on initial press** they come back at the brightness they had before instead.
- Every press that sets a brightness also sets the color temperature.

### The direction helper

The blueprint needs to remember whether the next press goes up or down. It keeps this in a toggle helper you create once: go to **Settings > Devices & services > Helpers**, select **Create helper** and pick **Toggle**, for example "Living room dimmer direction". Select it as the direction helper, and the blueprint switches it for you. Use one helper per automation.

Without a direction helper, the cycle only goes up. After the last step below full brightness, the next press turns the lights off, and the press after that starts again at the step percentage. With a step of 20% that is: 20%, 40%, 60%, 80%, off, 20% and so on.

### Limit extremes

With **Limit extremes** on, a press never turns the lights off or to full brightness. With a step of 20% the cycle bounces between 20% and 80%: 20%, 40%, 60%, 80%, 60%, 40%, 20%, 40% and so on. Use the off press to turn the lights off. Without a direction helper, the cycle jumps from the top back to the step percentage instead.

### Hold to dim

To change the brightness step by step while you hold the button, add `repeat` to the cycle event types and set the off event type to `long_release`, or leave it empty. Do not combine `repeat` with an off event of `long_press`, because the lights would then turn off while you hold the button.

On a Hue wall switch module, set the switch type to **Push button** in the Hue app. In the rocker and toggle modes the module only sends `initial_press` and `short_release`, so long presses do not work.

### Settings

| Setting | Default | What it does |
|---|---|---|
| Button | | The button entity that reports the presses. It must have an `event_type` attribute. Required. |
| Lights | | One or more lights to dim. Required. |
| Reference light | | One light whose brightness decides the next step. Usually one of the lights above. Required. |
| Direction helper | empty | A toggle helper that remembers the direction. Leave it empty for a cycle that only goes up. |

**Brightness cycle**

| Setting | Default | What it does |
|---|---|---|
| Step percentage | 20% | How much the brightness changes per press, from 1% to 100%. |
| Transition | 0 seconds | How long the lights take to fade to the new brightness, up to 10 seconds. |
| Color temperature | 4000 K | The color temperature set with each press, from 1500 K to 7000 K. |
| Limit extremes | off | Never turn the lights off or to full brightness with a cycle press. |
| Restore on initial press | off | When the lights are off, turn them on at their previous brightness instead of the step percentage. |

**Button events**

| Setting | Default | What it does |
|---|---|---|
| Cycle event types | `long_press` | The presses that move the brightness one step. Pick one or more of `initial_press`, `repeat`, `short_release`, `long_press` and `long_release`, or type another value your button sends. |
| Off event type | `short_release` | The press that turns the lights off, or on when they are off. Leave it empty to turn this off. |

**Debug**

| Setting | Default | What it does |
|---|---|---|
| Log to activity | off | Writes one line per press to the button's logbook, with the press type, the old and the new brightness and the direction. |

## Motion-triggered adaptive light

[![Import the Motion-triggered adaptive light blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fmm98%2Fha-blueprints%2Fblob%2Fmain%2Fblueprints%2Fautomation%2Fmm98%2Fmotion_ambient_nightmode_lights.yaml)

Turns a light on when motion is detected and the room is dark enough, and off again when the motion stops. At night it can use a lower brightness and a warmer color, so the light does not dazzle you. Other lights, switches or whole areas can follow along.

You need a motion sensor, a light sensor that measures the room's brightness in lux, and a light that can change its brightness and color temperature.

### How it works

- **Motion in a dark room.** When motion is detected and the light sensor shows less than the ambient threshold, the light turns on at the set brightness and color temperature. The additional targets turn on too.
- **Motion stops.** When there has been no motion for the grace period (2 minutes by default), the light and the additional targets turn off. New motion during that time starts the wait again.
- **The room gets bright.** When motion is detected while the room is at or above the threshold and the light is on, the light turns off.
- **Restart.** After Home Assistant restarts, for example after a power cut, the light and the additional targets turn off, so they do not stay on by mistake. You can turn this off.

### Night mode

With night mode on, the light turns on at the night brightness and night color temperature instead. Night mode follows a schedule, from 23:00 to 05:00 by default, and a schedule past midnight works. You can also select a night sensor instead, for example a toggle helper or a sleep mode switch: night mode is then on while that entity is on, and the schedule is not used.

### Blocking

Two optional blocking entities stop the light from turning on:

- **Blocking entity (on / true):** no light while this entity is on, for example a "Sleep mode" toggle helper.
- **Blocking entity (off / false):** no light while this entity is off, for example a "Motion lights enabled" toggle helper.

Blocking only stops the light from turning on. A light that is already on still turns off after the motion stops.

### Settings

| Setting | Default | What it does |
|---|---|---|
| Motion sensor | | The motion sensor that starts the automation. Required. |
| Motion-activated light | | The light that turns on and off. Required. |
| Additional targets | none | Other entities, devices or areas that turn on and off with the light, for example a second lamp or a switch. |
| Ambient light sensor | | The sensor that measures how bright the room is, in lux. Required. |

**Light**

| Setting | Default | What it does |
|---|---|---|
| Brightness | 20% | The brightness the light turns on at. |
| Transition | 2 seconds | How long the light takes to fade on or off, up to 60 seconds. |
| Ambient threshold | 4 lx | The light only turns on when the room is darker than this, from 0 to 50 lx. |
| Color temperature | 2500 K | The color temperature outside night mode, from 1500 K to 7000 K. |

**Night mode**

| Setting | Default | What it does |
|---|---|---|
| Night mode | off | Use the night brightness and color temperature at night. |
| Start time | 23:00 | When night mode starts. |
| End time | 05:00 | When night mode ends. |
| Night sensor | empty | An entity that turns night mode on while it is on. When set, the start and end times are not used. |
| Brightness | 10% | The brightness at night. |
| Color temperature | 2000 K | The color temperature at night. |

**Timing**

| Setting | Default | What it does |
|---|---|---|
| Motion grace period | 2 minutes | How long there must be no motion before the light turns off. |

**Blocking**

| Setting | Default | What it does |
|---|---|---|
| Blocking entity (on / true) | empty | The light does not turn on while this entity is on. |
| Blocking entity (off / false) | empty | The light does not turn on while this entity is off. |

**Restart**

| Setting | Default | What it does |
|---|---|---|
| Turn off light after restart | on | Turns the light and the additional targets off when Home Assistant starts. |

**Debug**

| Setting | Default | What it does |
|---|---|---|
| Log to activity | off | Writes a line to the logbook each time the automation acts, with the mode, brightness, color temperature, the light level and the threshold. |

## License

[MIT](LICENSE)
