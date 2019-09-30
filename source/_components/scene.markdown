---
title: "Scenes"
description: "Instructions on how to setup scenes within Home Assistant."
logo: home-assistant.png
ha_category:
  - Organization
ha_qa_scale: internal
ha_release: 0.15
---

You can create scenes that capture the states you want certain entities to be. For example, a scene can specify that light A should be turned on and light B should be bright red.

```yaml
# Example configuration.yaml entry
scene:
  - name: Romantic
    entities:
      light.tv_back_light: on
      light.ceiling:
        state: on
        xy_color: [0.33, 0.66]
        brightness: 200
  - name: Movies
    entities:
      light.tv_back_light:
        state: on
        brightness: 100
      light.ceiling: off
      media_player.sony_bravia_tv:
        source: HDMI 1
```

{% configuration %}
name:
  description: Friendly name of scene.
  required: true
  type: string
entities:
  description: Entities to control.
  required: true
  type: list
{% endconfiguration %}

As you can see, there are two ways to define the states of each `entity_id`:

- Define the `state` directly with the entity.
- Define a complex state with its attributes.

Scenes can be activated using the service `scene.turn_on` (there is no 'scene.turn_off' service).

```yaml
# Example automation
...
automation:
  trigger:
    platform: state
    entity_id: device_tracker.sweetheart
    from: 'not_home'
    to: 'home'
  action:
    service: scene.turn_on
    entity_id: scene.romantic
```
