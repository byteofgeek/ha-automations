# Home Assistant Zones Unleashed

**Six practical Home Assistant automations using location based zones.**

This collection accompanies my **Zones Unleashed** video on the [Byte of Geek YouTube channel](https://www.youtube.com/@ByteofGeek).

Home Assistant zones let you do much more than simply track whether someone is home or away. In the video, I demonstrate six practical examples of how zones can trigger notifications, track activities and manage your smart home automatically.

🎬 **[Watch Zones Unleashed on YouTube](https://youtu.be/5dvPNQrKVyg)**

## The Automations

| Automation | What it does |
|---|---|
| [Supermarket Shopping Reminder](supermarket-shopping-list.yaml) | Sends your outstanding shopping list items to your phone when you arrive at the supermarket. |
| [Leaving Work](leaving-work.yaml) | Sends a notification when someone leaves their workplace on a weekday. |
| [School Pickup Reminder](school-pickup.yaml) | Reminds you about school pickup when arriving at school between 2:00 pm and 4:30 pm, Monday to Friday. |
| [Commuting Mode](commuting-mode.yaml) | Enables a commuting helper when arriving at the train station and disables it when returning home. |
| [Gym Visit Tracking](gym-visit-tracking.yaml) | Counts gym visits and records the duration of your latest visit. |
| [Approaching Home](approaching-home.yaml) | Adjusts a heating target and welcome lighting helper based on household presence and an approaching-home zone. |

## Before You Start

These automations are examples of the configurations demonstrated in the video. They're designed to be adapted to your own Home Assistant installation.

You'll need:

- A working Home Assistant installation.
- Person entities configured for anyone whose location you want to track.
- A device tracker that provides location updates, such as the Home Assistant Companion App.
- The relevant zones configured in Home Assistant.
- Any additional helpers or integrations listed below.

**Important:** The Approaching Home example uses the `in_zones` person attribute introduced in Home Assistant 2026.10. You'll need a compatible version of Home Assistant for that example.

### Creating Zones

In Home Assistant, navigate to **Settings → Areas, labels & zones → Zones**.

Create the zones required for the automations you want to use.

| Zone | Entity ID |
|---|---|
| Home | `zone.home` |
| Supermarket | `zone.supermarket` |
| Work | `zone.work` |
| School | `zone.school` |
| Train Station | `zone.train_station` |
| Gym | `zone.gym` |
| Approaching Home | `zone.approaching_home` |

Home Assistant automatically creates a Home zone. The others need to be created manually.

Your zone entity IDs may differ depending on how you name them. Make sure the YAML references match your actual entity IDs.

For Approaching Home, create a larger zone surrounding your Home zone, so Home is contained within it.

## Automation Setup

### 1. Supermarket Shopping Reminder

**File:** [supermarket-shopping-list.yaml](supermarket-shopping-list.yaml)

When you enter the Supermarket zone, the automation checks whether your shopping list contains outstanding items.

If it does, it retrieves those items and sends a notification containing the list.

**Requirements:**
- Supermarket zone.
- A to-do list entity, such as `todo.shopping_list`.
- Home Assistant Companion App notifications.

**Customise:** Replace the person entity and mobile notification action. If you use a different to-do list, update all references to `todo.shopping_list`.

No notification is sent when the list has no outstanding items.

### 2. Leaving Work

**File:** [leaving-work.yaml](leaving-work.yaml)

Sends a notification when a person leaves the Work zone between Monday and Friday.

**Requirements:**
- Work zone.
- A person entity.
- A mobile notification action.

**Customise:** Replace the person entity, notification action and message.

The notification can be sent to a different household member's phone, rather than the phone belonging to the person leaving work.

### 3. School Pickup Reminder

**File:** [school-pickup.yaml](school-pickup.yaml)

Sends a reminder when entering the School zone between **14:00 and 16:30, Monday to Friday**.

**Requirements:**
- School zone.
- A person entity.
- A mobile notification action.

**Customise:** Change the pickup times, weekdays and notification message to suit your circumstances.

The time condition is checked when entering the zone. Arriving before the specified time won't trigger a reminder later unless you leave and re-enter the zone.

This example doesn't automatically account for school holidays or inset days.

### 4. Commuting Mode

**File:** [commuting-mode.yaml](commuting-mode.yaml)

Uses two zone triggers to manage a commuting mode helper:

- Entering the Train Station zone turns commuting mode on.
- Entering the Home zone turns commuting mode off.

**Requirements:**
- Train Station zone.
- Home zone.
- A Toggle helper named `Commuting Mode`, with entity ID `input_boolean.commuting_mode`.

**Create the helper:** Go to **Settings → Devices & services → Helpers → Create helper → Toggle**.

This automation only changes the helper's state. You can use that helper in other automations to manage heating, notifications or other behaviour while commuting.

Commuting mode remains enabled until you enter the Home zone, even if you visit other locations along the way.

### 5. Gym Visit Tracking

**File:** [gym-visit-tracking.yaml](gym-visit-tracking.yaml)

Tracks how many times you visit the gym and how long your most recent visit lasted.

When you enter the Gym zone, the automation increments a counter and records the arrival time.

When you leave, it calculates the elapsed time and stores the result in minutes.

**Requirements:**

| Helper type | Entity ID | Suggested configuration |
|---|---|---|
| Counter | `counter.gym_visits` | Initial value 0, step 1 |
| Date and/or time | `input_datetime.gym_arrival_time` | Enable both date and time |
| Number | `input_number.last_gym_visit_duration` | Minimum 0, maximum 1440, step 1, unit `min` |

Create these under **Settings → Devices & services → Helpers**.

**Things to be aware of:**

- Every entry into the Gym zone increments the counter.
- GPS fluctuations or briefly leaving and returning may result in additional visits.
- The duration includes all time spent inside the zone, not necessarily time spent exercising.
- A missing arrival event or an old saved arrival time can produce an inaccurate duration.

### 6. Approaching Home

**File:** [approaching-home.yaml](approaching-home.yaml)

This example combines household presence with an overlapping Approaching Home zone to manage heating and a welcome lighting helper.

It demonstrates how the `in_zones` attribute can detect membership of multiple zones, even when one zone overlaps another.

**Requirements:**
- Home zone.
- Approaching Home zone.
- Two person entities.
- Home Assistant 2026.10 or later.
- A Number helper and a Toggle helper.

**Required helpers:**

| Helper type | Entity ID | Suggested configuration |
|---|---|---|
| Number | `input_number.home_heating_target` | Minimum 5, maximum 30, step 0.5, unit °C |
| Toggle | `input_boolean.welcome_home_light` | Initially off |

**Heating behaviour:**

| Situation | Target temperature |
|---|---|
| Nobody home or approaching | 16°C |
| First person approaching home | 20°C |
| Either person home | 20°C |

**Welcome lighting behaviour:**

The welcome lighting helper turns on when the first person is inside the Approaching Home zone but hasn't yet entered the Home zone.

It turns off when that condition is no longer true.

**Important:** This automation doesn't directly control your heating system or lights. It updates two helpers that can be used by other automations to control those devices.

The example deliberately treats the two household members differently: the first person's approaching status controls the welcome lighting, while the second person's home status contributes to the heating decision.

You'll need to customise the person entities and helper IDs for your own household.

## How to Use the YAML Files

1. Open the YAML file for the automation you want.
2. Copy its contents.
3. In Home Assistant, navigate to **Settings → Automations & scenes**.
4. Create a new automation and select **Edit in YAML** from the automation editor's menu.
5. Replace the existing configuration with the example YAML.
6. Update the entity IDs, zones, notification actions and other settings as required.
7. Save the automation and test it.

**Don't forget:** The example entity IDs, such as `person.your_name` and `notify.mobile_app_your_phone`, are placeholders. They must be replaced with valid entities and actions from your own Home Assistant installation.

## Troubleshooting

If an automation doesn't behave as expected:

- Check that your person entity is updating correctly.
- Confirm that the zone entity IDs match those in the YAML.
- Make sure all required helpers and integrations exist.
- Check that your phone has granted the necessary location permissions.
- Review the automation traces in Home Assistant to see which triggers, conditions and actions executed.

GPS accuracy and location update frequency can affect how quickly zone automations respond.

## Support Byte of Geek

If you've found these examples useful and would like to support the work that goes into my videos and tutorials, you can:

[![Buy Me a Coffee](https://img.shields.io/badge/Buy_Me_a_Coffee-Support_the_channel-FFDD00?style=flat)](https://www.buymeacoffee.com/byteofgeek)

Make a one-off contribution to support the channel.

[![YouTube Membership](https://img.shields.io/badge/YouTube-Become_a_Member-FF0000?style=flat)](https://www.youtube.com/channel/UCU2nsuXb1xP_5DSgsik8J0Q/join)

Support the channel through a monthly YouTube membership.

Watching, commenting and subscribing are also great ways to support the channel!

## Disclaimer

These configurations are provided for educational purposes and should be reviewed and adapted before use. Always test automations carefully, particularly those affecting heating or other physical devices.

[← Back to all Home Assistant automations](../README.md)