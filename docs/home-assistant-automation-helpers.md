# Home Assistant Examples

These examples assume that [docs/home-assistant-DGB-config-automation.md](../docs/home-assistant-DGB-config-automation.md) has been run and that Home Assistant created entities from the MQTT discovery configuration. Replace the entity IDs and notification target with the names used by your installation.

## Doorbell notification

```yaml
alias: 'Huis: Deur bel bericht'
description: Stuur bericht wanneer de deurbel gaat.
triggers:
  - trigger: state
    entity_id:
      - binary_sensor.deurbel_deurbel_detectie
    to:
      - 'on'
actions:
  - data:
      title: Deurbel gaat
      message: Er staat iemand voor de deur
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 0.3
    action: notify.mobile_app_iphone_jeroen
mode: single
```

## Temporarily silence the doorbell

This turns the doorbell off for two hours and then restores it. There is a backup that turns the bell on when Home Assistant shuts down. 

```yaml
alias: Huis - Schakel deurbel tijdelijk uit
description: Schakel de deurbel een aantal uur uit volgens nummer input
triggers:
  - trigger: state
    entity_id:
      - switch.deurbel_deurbel_aan_uit
    to:
      - 'off'
    id: uit
  - trigger: homeassistant
    event: shutdown
    id: aan
  - trigger: homeassistant
    event: start
    id: aan
actions:
  - if:
      - condition: trigger
        id:
          - uit
    then:
      - action: switch.turn_off
        target:
          entity_id: switch.deurbel_deurbel_aan_uit
        data: {}
      - delay: '02:00:00'
  - action: switch.turn_on
    target:
      entity_id: switch.deurbel_deurbel_aan_uit
    data: {}
mode: restart
```

