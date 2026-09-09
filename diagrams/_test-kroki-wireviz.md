Wireviz yaml code to generate Wiring diagram for subsystem on the MKcontroller. Use https://marketplace.visualstudio.com/items?itemName=pomdtr.markdown-kroki to preview the diagram. Optionally you can run [Kroki server](https://docs.kroki.io/kroki/setup/install/) in docker. 

**Water 1 (Hard) & Water 2 (Soft - optional):**
| Pin | Function | Wiring |
|-----|----------|--------|
| 1 | GND | Black wire |
| 2 | +5V Power | Red wire |
| 3 | Pulse Signal | Yellow wire → GPIO 23 (water 1) / GPIO 24 (water 2) |

```kroki-wireviz

connectors:
  PI Zero W2:
    pins: [GND, 5V, GPIO23]

  GX12:
    type: GX12-3
    pins: [1, 2, 3]

  Water sensor:
    pins: [GND, 5V, SIG]

cables:
  W_PI_GX12:
    wirecount: 3
    colors: [BK, RD, YE]
    show_name: False
    show_wirecount: False

  W_GX12_SENSOR:
    wirecount: 3
    colors: [BK, RD, YE]
    show_name: False
    show_wirecount: False

connections:
  - - PI Zero W2: [GND, 5V, GPIO23]
    - W_PI_GX12: [1, 2, 3]
    - GX12: [1, 2, 3]

  - - GX12: [1, 2, 3]
    - W_GX12_SENSOR: [1, 2, 3]
    - Water sensor: [GND, 5V, SIG]
    


tweak:
  override:
    graph:
      ranksep: "0.5"
    W_PI_GX12:
      shape: none
      fontsize: "1"
      fontcolor: white
      
    W_GX12_SENSOR:
      shape: none
      fontsize: "1"
      fontcolor: white
   ```   