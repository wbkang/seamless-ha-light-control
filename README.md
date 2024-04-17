
## Overview

This is a blueprint for a seamless motion sensor-based wall switch driven light control.

It has the following features:
1. When motion is detected, turn on the light for a specific time. Extend the time whenever motion is detected.
1. When the light is turned on, turn the light off after a timeout.
1. When the light is turned off manually, do not trigger the motion detection light for a while (customizable).

## Instructions

### Install
1. Git clone this repo under blueprints/automation/

### Prerequisite

1. Create a timer for your light
1. Create a suppression timer for your light. This is used to make sure the motion sensor does not trigger the light again when a human turns it off.
1. Optionally, create a binary_sensor named `binary_sensor.always_off` which is always off. This is necessary because the blueprint always needs A sensor for a trigger. 

### Create Automations

Create 4 automations each of the blueprint element.

## What do they do?

- The main timer is used to drive the lights. When it's active, the light turns on, and vice versa.
- The suppression timer is used to suppress the lights from coming on, if the light was turned off manually from the wall switch.

Blueprints
- light_off_event: Used to bind a manual light off event to reset the main timer, and start the suppression timer.
- light_on_event: Used to bind a manual light on event to start the main timer, and reset the suppression timer.
- motion_sensor_to_timer: Binds the motion sensor event into the main timer, which will eventually trigger the light on.
- timer_event_to_light: Binds the main light timer state to light.


You can hook up multiple motion sensors, and multiple lights for each group.

## Example usage

binary_sensor.always_off definition:

```
binary_sensor:
  - platform: template
    sensors:
      always_off:
        friendly_name: always_off
        value_template: "{{ 'off' }}"
```

Timer definitions:

```
timer:
  dining_room_timer:
    name: "Dining Room Light Timer"
    duration: 00:20:00
    restore: true
  dining_room_suppression_timer:
    name: "Dining Room Suppression Timer"
    duration: 00:10:00
    restore: true
```

```
- id: '1679614828756'
  alias: 'Dining room: Light on event'
  use_blueprint:
    path: seamless-light-control/light_on_event.yaml
    input:
      light_timer: timer.dining_room_timer
      suppression_timer: timer.dining_room_suppression_timer
      target_light: light.dining_room_light
- id: '1679617833881'
  alias: 'Dining room: Motion  Sensor to Light Timer (4-12)'
  use_blueprint:
    path: seamless-light-control/motion_sensor_to_timer.yaml
    input:
      motion_sensor_1: binary_sensor.dining_room_motion
      motion_sensor_2: binary_sensor.always_off
      active_time_before: '23:59:59'
      active_time_after: 04:00:00
      light_timer_1: timer.dining_room_timer
      light_timer_2: timer.living_room_timer
      suppression_timer: timer.dining_room_suppression_timer
- id: '1679617900892'
  alias: 'Dining room: Timer to light'
  use_blueprint:
    path: seamless-light-control/timer_event_to_light.yaml
    input:
      light_timer: timer.dining_room_timer
      target_light: light.dining_room_light
- id: '1679618708885'
  alias: 'Dining room: light off event'
  use_blueprint:
    path: seamless-light-control/light_off_event.yaml
    input:
      light_timer: timer.dining_room_timer
      suppression_timer: timer.dining_room_suppression_timer
      target_light: light.dining_room_light

```