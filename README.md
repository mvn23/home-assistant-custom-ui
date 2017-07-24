# Custom UI elements for https://home-assistant.io

  * [Changelog](#changelog)
  * [Usage](#usage)
  * [Features available for all domains](#features-available-for-all-domains)
    + [Context-aware attributes](#context-aware-attributes)
    + [Badges in state cards](#badges-in-state-cards)
  * [Features available for almost all domains](#features-available-for-almost-all-domains)
      - [You can always show the last-changed text](#you-can-always-show-the-last-changed-text)
  * [Features available for light, cover, "plain", and "toggle" cards](#features-available-for-light-cover-plain-and-toggle-cards)
      - [You can hide the control altogether](#you-can-hide-the-control-altogether)
      - [You can add extra data below the entity name [Requires HA 0.43+]](#you-can-add-extra-data-below-the-entity-name-requires-ha-043)
      - [Add badge to the state card [Requires HA 0.42+]](#add-badge-to-the-state-card-requires-ha-042)
  * [Features available for light and cover domains only](#features-available-for-light-and-cover-domains-only)
      - [If there is not enough horizontal space the mode is set by `state_card_mode` parameter](#if-there-is-not-enough-horizontal-space-the-mode-is-set-by-state_card_mode-parameter)
      - [If the slider got moved to a new line it will be 200 px wide.](#if-the-slider-got-moved-to-a-new-line-it-will-be-200-px-wide)
      - [The slider behavior is controlled by `slider_theme` dictionary.](#the-slider-behavior-is-controlled-by-slider_theme-dictionary)
  * [Complete example](#complete-example)
  * [Known issues](#known-issues)

## Changelog

#### 2017-07-06
* Fix bug introduced on 20170701 which caused slider to do nothing.

[Full Changelog](CHANGELOG.md)

## Usage
Place [state-card-custom-ui.html](state-card-custom-ui.html?raw=true) and [state-card-custom-ui.html.gz](state-card-custom-ui.html.gz?raw=true) in `~/.homeassistant/www/custom_ui/` dir.  

In the `customize:` section of `configuration.yaml` put `custom_ui_state_card: custom-ui` for the relevant entities / domains.

For example:
```yaml
homeassistant:
  customize_glob:
    light.*:
      custom_ui_state_card: custom-ui
    cover.*:
      custom_ui_state_card: custom-ui
```

Note that yaml keys can't start with an asterix. Use quotes in that case:
```yaml
customize_glob:
  "*.*":
    custom_ui_state_card: custom-ui
```
## Features available for all domains

### Context-aware attributes
![context_aware](https://cloud.githubusercontent.com/assets/5478779/26284053/45fbc000-3e3b-11e7-8d4a-56ef0d5e6c60.png)
You can use context-aware attributes to give different names for the same entity in different groups or views.
For example if you have a *Yard Light* and a *Yard Sensor* in a group named *Yard*, you could name the entities as *Light* and *Sensor* in the group only by using context-aware `friendly_name` attribute. This will also work in views (`view: yes` groups). In order to rename an entity in the default view, use `deafult_view` view name (even if you didn't define such a view).

Example:
```yaml
homeassistant:
  customize_glob:
    "*.*":
      custom_ui_state_card: custom-ui  
    light.yard_light:
      friendly_name: Yard Light
      group:
        group.yard:
          friendly_name: Light
    sensor.yard_sensor:
      friendly_name: Yard Sensor
      group:
        group.yard:
          friendly_name: Sensor

group:
  yard:
    entities:
      - light.yard_light
      - sensor.yard_sensor
```

#### Context-aware hide
In case you want a device to be a member of a group but not *show* in the group - use context-aware `hidden` attribute.
Unlike regular `hidden: true` which hides the device in all views, context-aware `hidden: true` will hide the devices in specified groups only.
```yaml
homeassistant:
  customize:
  ...
    light.yard_light:
      group:
        group.yard:
          hidden: true

group:
  yard:
    entities:
      - light.yard_light
      ...
```

#### Other uses
Context-aware attributes also work for custom attributes, like `hide_control`, `show_last_changed`, and others.

### Badges in state cards
![badges](https://cloud.githubusercontent.com/assets/5478779/26284132/b4a2dbe6-3e3c-11e7-9bb5-0441d30342bf.png)

If you like badges, you can now put them in the state cards. This also works for domains that are usually not used as a badge. Lights for example.
There are 3 ways to put badges in a state card.

1) Create a dedicated group of devices you want to display as badges and apply `state_card_mode: badges` to it. Note that this group must be in another group. The example below will show 2 sensors as badges in outer_group's card.
```yaml
homeassistant:
  customize_glob:
    "*.*":
      custom_ui_state_card: custom-ui  
    group.inner_group:
      state_card_mode: badges

group:
  inner_group:
    entities:
      - sensor.door_sensor
      - sensor.yard_sensor
  outer_group:
    entities:
      - group.inner_group
      *all other devices of outer_group*
```

2) If you already have a group, *part* of which you want to display as badges *inside another group* - use `badges_list` to filter badge wannabe entities. In the previous example, if you wanted to show only `sensor.door_sensor` as a badge in outer_group:
```yaml
...
    group.inner_group:
      state_card_mode: badges
      badges_list:
        - sensor.door_sensor
group:
  inner_group:
     ...
  outer_group:
    ...
...
```

3) Creating a dedicated group has a downside that the group will also show in the UI as whole in the default_view. To prevent that, you can make the group include itself. In the following example `inner_group` and `outer_group` are the same group:
```yaml
homeassistant:
  customize_glob:
    "*.*":
      custom_ui_state_card: custom-ui  
    group.my_group:
      state_card_mode: badges

group:
  my_group:
    entities:
      - sensor.door_sensor
      - sensor.yard_sensor
      - group.my_group
      *all other devices of outer_group*
```
If you use this example as-is you will notice that all of your devices in the group appear both as regular state cards and as badges. To limit badges to the door/yard sensors only use `badges_list` from Example 2. To hide door/yard sensor cards (but leave them as badges) use the [Context-aware hide attribute](#context-aware-attributes) feature.  
Full example:
```yaml
homeassistant:
  customize_glob:
    "*.*":
      custom_ui_state_card: custom-ui  
    group.my_group:
      state_card_mode: badges
      badges_list:
        - sensor.door_sensor
        - sensor.yard_sensor
    sensor.door_sensor:
      group:
        group.my_group:
          hidden: true
    sensor.yard_sensor:
      group:
        group.my_group:
          hidden: true

group:
  my_group:
    entities:
      - sensor.door_sensor
      - sensor.yard_sensor
      - light.mylight
      - group.my_group
```

## Features available for almost all domains.

The following is supported for all state cards except `configurator`

#### You can always show the last-changed text
![show_last_changed](https://cloud.githubusercontent.com/assets/5478779/24838935/37b90bf8-1d5a-11e7-9e28-970740ba2fa8.png)

Use `show_last_changed: true`

Note that if you use the [extra_data_template](#you-can-add-extra-data-below-the-entity-name-requires-ha-043) below it will take precedence over `show_last_changed`

## Features available for light, cover, "plain", and "toggle" cards.

The next features are available for 4 types of cards:
* Light
* Cover,
* "Plain" i.e. card with icon, name, and state (not including binary_sensor)
* "Toggle" i.e. card with icon, name, and toggle.

#### You can hide the control altogether
![hide_control](https://cloud.githubusercontent.com/assets/5478779/24772031/8a7d546e-1b18-11e7-935a-4360eeb9ebc8.png)

Use `hide_control: true` to hide the control (toggle / cover buttons) altogether.

#### You can add extra data below the entity name [Requires HA 0.43+]
![extra_data](https://cloud.githubusercontent.com/assets/5478779/24772032/8a7e90e0-1b18-11e7-9b3e-e36b56ef2417.png)

Use `extra_data_template` to add extra data below the entity name. The format is a string where `{attribute_name}`will be replaced by the attribute value.
For example `{power_consumption}W` will parse as `27W` if the value of `power_consumption` is 27.

You can add an attribute value conditionally if it isn't equal to some constant. For example `{power_consumption!=0}W` to only add power consumption if it is not zero.

#### Add badge to the state card [Requires HA 0.42+]
![extra_badge](https://cloud.githubusercontent.com/assets/5478779/24772030/8a7cc4ea-1b18-11e7-9313-f7654ffb0c71.png)

Instead of using a grey text below the enity name you can add a sensor-like badge. There are two ways to do that:
1) Specify a real sensor by entity ID:
```yaml
extra_badge:
  entity_id: sensor.my_sensor
```
2) Make a fake sensor from entity's attribute:
```yaml
extra_badge:
  attribute: power_consumption
  unit: W
```

In both cases you can specify a blacklist of badge "states", when you don't want to see the badge.
```yaml
extra_badge:
  entity_id: sensor.my_sensor
  blacklist_states: 0
```

## Features available for light and cover domains only

If there is enough space the card will have icon+name on the left, optional slider in the middle and toggle on the right:

![cover](https://cloud.githubusercontent.com/assets/5478779/23921980/4eab7978-0909-11e7-8058-ad17a52d93c3.png)

![wide](https://cloud.githubusercontent.com/assets/5478779/23335593/e344048e-fbc0-11e6-81fd-85466a6b98b2.png)

#### If there is not enough horizontal space the mode is set by `state_card_mode` parameter
![medium](https://cloud.githubusercontent.com/assets/5478779/23335594/e909eee2-fbc0-11e6-8429-8648b89d6d13.png) ![narrow](https://cloud.githubusercontent.com/assets/5478779/23335595/eceaa92a-fbc0-11e6-9dff-018585f60ff0.png)

| `state_card_mode` value | description |
| --- | --- |
| break-slider-toggle | Move the slider and the toggle together to a second line. | 
| single-line | Never use more than one line. Shrink the name and the slider. |
| break-slider | Move slider to second line. Leave toggle on the first line.|
| hide-slider | Hide the slider.|
| no-slider (default) | Never show the slider even if there is enough space. |

#### If the slider got moved to a new line it will be 200 px wide.
Use `stretch_slider` attribute to make it strech to all available space.

#### The slider behavior is controlled by `slider_theme` dictionary.
In that dictionary the following optional fields are available:

| field | default | description |
| --- | --- | --- |
| min | 0 | Minimum slider value |
| max | 255 for light, 100 for cover | Maximum slider value |
| pin | False | Display numeric value when moving the slider |
| off_when_min | True | Whether to turn the light *off* when moving the slider to the mininmum value if that value is not 0 |
| report_when_not_changed | True | Whether to send the light-controlling command if the slider was returned to the initial position. I.e. you movied the slider and then changed your mind |

## Complete example
```yaml
homeassistant:
  customize:
    light.bedroom:
      custom_ui_state_card: custom-ui
      state_card_mode: break-slider
      stretch_slider: true
      extra_data_template: "{power_consumption!=10}W"
      hide_control: false
      show_last_changed: false
      slider_theme:
        min: 10
        max: 200
        pin: true
        off_when_min: false
        report_when_not_changed: false
      extra_badge:
        entity_id: sensor.my_sensor  # Will take precedence over attribute and unit below.
        attribute: power_consumption
        unit: W
        blacklist_states: 0
        
```


## Known issues
Custom UI on iOS 10 / Mac Safari 10 requires HA 0.41
