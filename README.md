# Prometheus Exporter for metrics received via MQTT

This is a fork of [Prometheus Exporter](https://github.com/Svedrin/mqtt-pushgateway).

This is listening to some interesting MQTT topics and then performs requests to
your pushgateway API via HTTP, while the original one provides a full push
gateway for MQTT, where logs are scraped by Prometheus.

Messages that can be parsed as JSON will log a unique metric per key:value pair
using a 'virtual' topic of `topic/key`.

Caveat: Only float values are supported. Anything else will be ignored.

## Features

* Topics can be matched against a set of regular expressions to convert parts of
  their names into Prometheus keywords. Thus, a topic name such as
  `sensor/garage/temperature` when matched using regex
  `sensor/(?P<sensor_name>\w+)/(?P<__metric__>\w+)` would be exported to
  Prometheus as metric
  `temperature{mqtt_topic="sensor/garage/temperature",sensor_name="garage"}
  29.3`,
* Each metric is accompanied by an `mqtt_data_age` metric, that tells us when
  the last update occurred: `mqtt_data_age{mqtt_topic="sensor/garage/temperature",sensor_name="garage",metric="temperature"} 7`,
  by setting a threshold on this value, you can detect and alert on sensors
  being broken somehow and not sending updates anymore.
* Topics that were matched in a subscription pattern can be hidden from the
  result through a topic configuration.

* JSON messages record each key:value pair as a unique metric, eg: the following
  payload sent on topic `zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM`:

    ```json
    {"temperature":29.02,"linkquality":34,"humidity":55.58,"battery":100,"voltage":3005,"status":"online"}
    ```

  would expose the following metrics:

    ```
    temperature{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/temperature"} 29.020000
    linkquality{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/linkquality"} 34.000000
    humidity{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/humidity"} 55.580000
    battery{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/battery"} 100.000000
    voltage{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/voltage"} 3005.000000
    status{mqtt_topic="zigbee2mqtt/sensor/lounge/xiaomi/WSDCGQ01LM/status",status="online"} 1.0
    ```

## Installation

* `pip3 -r requirements.txt`.
* Copy `config.example.toml` to `config.toml` and adapt it to your needs.
* Run `mqtt_pushgateway.py`. (See [mqtt-pushgateway.service](mqtt-pushgateway.service))

## Docker

You can also use Docker:

* `docker build -t ktos/prometheus-mqtt:latest .`
* `docker run -v config.toml:/config/config.toml ktos/prometheus-mqtt:latest`
