# Home Assistant Examples

This automations will publish the door bell and water meter DGB configuration when Home Assistant has started. Dispite the retained status of the message, It seems wise it republish each restart. 

## Roller door configuration

```yaml
alias: Configureer DGB services
mode: single
description: ""
triggers: 
  - trigger: switch.turned_on
    target:
      entity_id: input_boolean.hass_opgestart
    options:
      behavior: each
      for: '00:00:00'
conditions: []
actions:
  - alias: MK-controller-bel
    action: mqtt.publish
    metadata: {}
    data:
      qos: '2'
      retain: true
      topic: config/MK-controller/devices/deurbel
      payload: |
        {
          "Devices": [
            {
              "EntityInfo": {
                "component": "binary_sensor",
                "name": "Deurbel detectie",
                "qos": 2,
                "unique_id": "192-168-70-28-binary-sensor-deurbel",
                "device_class": "sound",
                "device": {
                  "name": "Deurbel",
                  "model": "Knop",
                  "identifiers": "lvb11-deurbel",
                  "configuration_url": "http://192-168-70-28",
                  "suggested_area": "meterkast"
                }
              }
            },
            {
              "EntityInfo": {
                "component": "switch",
                "name": "Deurbel uit/aan",
                "qos": 2,
                "unique_id": "192-168-70-28-switch-deurbel-aan-uit",
                "device": {
                  "name": "Deurbel",
                  "model": "Knop",
                  "identifiers": "lvb11-deurbel",
                  "configuration_url": "http://192-168-70-28",
                  "suggested_area": "meterkast"
                }
              }
            }
          ],
          "Pins": [
            {
              "PinInfo": {
                "pin": 26,
                "unique_id": "deurbel_uit_aan_relay_pin_26",
                "ptype": "out",
                "initial": 0,
                "value": 1,
                "active_state": false
              }
            },
            {
              "PinInfo": {
                "pin": 25,
                "unique_id": "deurbel_sensor_pin_25",
                "ptype": "in",
                "pull_up": true
              }
            }
          ],
          "state_initialization": {
            "persist_action": [
              {
                "unique_id": "deurbel_uit_aan_relay_pin_26",
                "call": ["set_state"]
              },
              {
                "unique_id": "192-168-70-28-switch-deurbel-aan-uit",
                "call": ["set_state"]
              }
            ]
          },
          "Bindings": [
            {
              "BindInfo": {
                "pin_25_to_deurbel_detectie": {
                  "p_on": {
                    "all": [
                      {
                        "m": {
                          "$and": [
                            {"unique_id": "deurbel_sensor_pin_25"},
                            {"value": 1}
                          ]
                        }
                      }
                    ],
                    "run": {
                      "action": {
                        "unique_id": "192-168-70-28-binary-sensor-deurbel",
                        "call": "on"
                      }
                    }
                  },
                  "p_off": {
                    "all": [
                      {
                        "m": {
                          "$and": [
                            {"unique_id": "deurbel_sensor_pin_25"},
                            {"value": 0}
                          ]
                        }
                      }
                    ],
                    "run": {
                      "action": {
                        "unique_id": "192-168-70-28-binary-sensor-deurbel",
                        "call": "off"
                      }
                    }
                  }
                }
              }
            },
            {
              "BindInfo": {
                "pin_26_deurbel_on_off": {
                  "p_on": {
                    "all": [
                      {
                        "m": {
                          "$and": [
                            {"unique_id": "192-168-70-28-switch-deurbel-aan-uit"},
                            {"value": "ON"}
                          ]
                        }
                      }
                    ],
                    "run": {
                      "action": {
                        "unique_id": "deurbel_uit_aan_relay_pin_26",
                        "call": "on"
                      }
                    }
                  },
                  "p_off": {
                    "all": [
                      {
                        "m": {
                          "$and": [
                            {"unique_id": "192-168-70-28-switch-deurbel-aan-uit"},
                            {"value": "OFF"}
                          ]
                        }
                      }
                    ],
                    "run": {
                      "action": {
                        "unique_id": "deurbel_uit_aan_relay_pin_26",
                        "call": "off"
                      }
                    }
                  }
                }
              }
            }
          ]
        }
  - alias: MK-controller-water
    action: mqtt.publish
    metadata: {}
    data:
      qos: '2'
      retain: true
      topic: config/MK-controller/devices/water
      payload: |-
        {
          "Devices": [
            {
              "EntityInfo": {
                "component": "sensor",
                "name": "Hard water volume",
                "qos": 0,
                "unique_id": "192-168-70-28-sensor-hard-water-volume",
                "device_class": "water",
                "unit_of_measurement": "m³",
                "state_class": "total_increasing",
                "device": {
                  "name": "Water meter",
                  "model": "YF-B6",
                  "identifiers": "lvb11-water-meter",
                  "configuration_url": "https://www.tinytronics.nl/nl/sensoren/vloeistof/yf-b6-water-flow-sensor-met-temperatuur-sensor-messing-g3-4",
                  "suggested_area": "meterkast"
                }
              }
            },
            {
              "EntityInfo": {
                "component": "sensor",
                "name": "Hard water stroom",
                "qos": 0,
                "unique_id": "192-168-70-28-sensor-hard-water-stroom",
                "device_class": "volume_flow_rate",
                "unit_of_measurement": "L/min",
                "state_class": "measurement",
                "device": {
                  "name": "Water meter",
                  "model": "YF-B6",
                  "identifiers": "lvb11-water-meter",
                  "configuration_url": "https://www.tinytronics.nl/nl/sensoren/vloeistof/yf-b6-water-flow-sensor-met-temperatuur-sensor-messing-g3-4",
                  "suggested_area": "meterkast"
                }
              }
            }
          ],
          "Pins": [
            {
              "PinInfo": {
                "pin": 23,
                "unique_id": "hard-water-stroom-pin-23",
                "ptype": "count",
                "pull_up": true,
                "when_deactivated": false,
                "scaling_factor": 396000
              }
            }
          ],
          "state_initialization": {
            "persist_action": [
              {
                "unique_id": "hard-water-stroom-pin-23",
                "call": ["set_state"]
              },
              {
                "unique_id": "192-168-70-28-sensor-hard-water-volume",
                "call": ["set_state"]
              }
            ]
          },
          "Bindings": [
            {
              "BindInfo": {
                "pin_23_to_hard_water_meter-volume": {
                  "p_on": {
                    "count": 500,
                    "all": [
                      {
                        "m": {
                          "unique_id": "hard-water-stroom-pin-23"
                        }
                      }
                    ],
                    "run": {
                      "action": {
                        "unique_id": "192-168-70-28-sensor-hard-water-volume",
                        "call": "set_state",
                        "args": [
                          {
                            "name": "state",
                            "value": {
                              "$ref": "m.value"
                            }
                          }
                        ]
                      }
                    }
                  }
                }
              }
            }
          ]
        }

```
