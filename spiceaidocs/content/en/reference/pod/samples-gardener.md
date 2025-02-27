---
type: docs
title: "Gardener"
linkTitle: "Example - Gardener"
weight: 60
---

From: https://github.com/spiceai/samples/blob/trunk/gardener/README.md

```yaml
name: gardener
params:
  epoch_time: 1612557000
  granularity: 10m
  interval: 1h
  period: 720h
dataspaces:
  - from: sensors
    name: garden
    fields:
      - name: temperature
      - name: moisture
    data:
      connector:
        name: file
        params:
          path: data/garden_data.csv
      processor:
        name: csv

actions:
  - name: close_valve
  - name: open_valve_half
  - name: open_valve_full

training:
  rewards:
    - reward: close_valve
      with: |
        # Reward keeping moisture content above 25%
        if new_state.sensors_garden_moisture > 0.25:
          reward = 200

        # Penalize low moisture content depending on how far the garden has dried out
        else:
          reward = -100 * (0.25 - new_state.sensors_garden_moisture)

          # Penalize especially heavily if the drying trend is continuing (new_state is drier than prev_state)
          if new_state.sensors_garden_moisture < prev_state.sensors_garden_moisture:
            reward = reward * 2

    - reward: open_valve_half
      with: |
        # Reward watering when needed, more heavily if the garden is more dried out
        if new_state.sensors_garden_moisture < 0.25:
          reward = 100 * (0.25 - new_state.sensors_garden_moisture)

        # Penalize wasting water
        # Penalize overwatering depending on how overwatered the garden is
        else:
          reward = -50 * (new_state.sensors_garden_moisture - 0.25)

    - reward: open_valve_full
      with: |
        # Reward watering when needed, more heavily if the garden is more dried out
        if new_state.sensors_garden_moisture < 0.25:
          reward = 200 * (0.25 - new_state.sensors_garden_moisture)

        # Penalize wasting water more heavily with valve fully open
        # Penalize overwatering depending on how overwatered the garden is
        else:
          reward = -100 * (new_state.sensors_garden_moisture - 0.25)
```
