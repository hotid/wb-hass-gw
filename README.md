# Wiren Board - Home Assistant geteway

Gateway to connect [Wiren Board](https://wirenboard.com/) to [Home Assistant](http://home-assistant.io) 
with [MQTT discovery](https://www.home-assistant.io/docs/mqtt/discovery/s).

* All devices connected to WirenBoard should be added to HomeAssistant automatically. 
* You don't need to write manually configuration for each sensor/switch/etc...
* You don't need manually configure MQTT bridge. All needed messages between Wiren Board and Home Assistant will be forwarded by the gateway.
* Device availability is also tracked based on [meta/error](https://github.com/wirenboard/homeui/blob/master/conventions.md) properties  


## Installing

#### **wb-hass-gw** can be installed directly to Wiren Board:

```shell script
apt install pip3
pip3 install virtualenv
cd /opt
git clone https://github.com/Hacker-CB/wb-hass-gw
cd wb-hass-gw
virtualenv -p python3 .venv
source .venv/bin/activate
pip install -r requirements.txt
```
### Configuring wb-hass-gw

```yaml
wirenboard:
  broker_host: [YOUR HASS INSTANCE]

homeassistant:
  broker_host: 127.0.0.1
  topic_prefix: my_wirneboard/
  entity_prefix: WB 1
```

### Configuring Home Assistant

```yaml
mqtt:
  broker_host: [YOUR MQTT BROKER]  # Remove if you want to use builtin-in MQTT broker 
  discovery: true
  birth_message:
    topic: 'hass/status'
    payload: 'online'
  will_message:
    topic: 'hass/status'
    payload: 'offline'
```

NOTE: birth_message/will_message are needed to resend all devices after Home Assistant restarts. (like in [zigbe2mqtt](https://www.zigbee2mqtt.io/integration/home_assistant.html))

### Run 
```shell script
python wb-hass-gw.py -c /etc/wb-hass-hw.yaml
```


## Full config with default values
```yaml
general:  
  loglevel: INFO # One of INFO/WARNING/ERROR/FATAL
  
  wirenboard: 
    broker_host: 
    broker_port: 1883
    username:
    password:
    client_id: 'wb-hass-gw'
    topic_prefix: ''
  
  homeassistant:
    broker_host: 
    broker_port: 1883
    username:
    password:
    client_id: 'wb-hass-gw'
    topic_prefix:
    entity_prefix: ''
    discovery_topic: 'homeassistant'
    status_topic: 'hass/status'
    status_payload_online: 'online'
    status_payload_offline: 'offline'
```


## Supported Wiren Board controls

All Wiren Board types are documented [Wiren Board MQTT Conventions](https://github.com/wirenboard/homeui/blob/master/conventions.md)

#### Generic types
| Wiren Board             |  Home Assistant |
|-------------------------|-----------------|
| switch                  |  switch         |         
| switch(read_only)       |  binary_sensor  |         
| alarm                   |  binary_sensor  |        
| pushbutton              |  binary_sensor  |             
| range                   |  -              |                    
| range(read_only)        |  sensor         |        
| rgb                     |  -              |                  
| text                    |  sensor         |       
| value                   |  sensor         |        

#### Special types

| Wiren Board           |  Home Assistant |
|-----------------------|-----------------|
| temperature           |  sensor         |                                
| rel_humidity          |  sensor         |                                
| atmospheric_pressure  |  sensor         |                                        
| rainfall              |  sensor         |                            
| wind_speed            |  sensor         |                                
| power                 |  sensor         |                        
| power_consumption     |  sensor         |                                    
| voltage               |  sensor         |                            
| water_flow            |  sensor         |                                
| water_consumption     |  sensor         |                                    
| resistance            |  sensor         |                                
| concentration         |  sensor         |                                
| heat_power            |  sensor         |                                
| heat_energy           |  sensor         |                                

## TODO

* Add support for `range`
* Add support for `rgb`
* Add debounce policy to prevent flood on analog sensors. For example. Wiren Board can poll MODBUS every 10ms, but we don't need so often updates on analog sensors. 
This can be only useful on digital inputs to handle click events.
* Need to delete entities after they will be disappeared in Wiren Board.
* Add device information if available (version, serial)

## Known issues

* There is no built-in entity to represent Wiren Board `range` type as editable entity. 
I have tried to use `cover` and `light` for it, but it not looked nice. Suggestions are welcome.