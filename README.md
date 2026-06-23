# Homebridge Device Lab

Create Homebridge accessories backed by HTTP, HTTPS, or MQTT data sources, with JSONata-powered characteristic mapping.

Device Lab is a fork of `homebridge-http-sensors-switches` that is moving toward a broader virtual-device toolkit: JSON-backed accessories first, synthetic/random values next, and fewer one-off plugins over time.

## Current Capabilities

- HTTP and HTTPS JSON polling for sensors, switches, lights, outlets, fans, doors, windows, window coverings, valves, and common environmental sensors.
- MQTT-backed state updates and control for supported device types.
- Shared polling groups so multiple accessories can read from the same HTTP JSON payload.
- JSONata expressions anywhere a JSON parameter field is used.
- Side-by-side installation with the original plugin through the `DeviceLab` platform alias.

## Removed From This Fork

- Discord webhook notifications.
- Built-in per-sensor transform functions.

Use JSONata expressions instead of built-in transforms. For example, HomeKit requires ambient light values above zero, so map a zero lux reading like this:

```json
"paramNameCurrentAmbientLightLevel": "lastData.solarradiation <= 0 ? 0.0001 : lastData.solarradiation"
```

## Platform Alias

Use this platform name in `config.json`:

```json
"platform": "DeviceLab"
```

This allows Device Lab to run alongside the original plugin:

```json
"platform": "HttpSensorsAndSwitches"
```

## Example Configuration

```json
{
  "name": "Device Lab",
  "platform": "DeviceLab",
  "_bridge": {
    "username": "0E:36:A3:C1:48:4F",
    "port": 55530,
    "name": "Device Lab"
  },
  "devices": [
    {
      "deviceType": "LightSensor",
      "enableLogging": false,
      "deviceID": "devicelab-ambient-light-lux",
      "deviceName": "Device Lab Ambient LUX",
      "deviceManufacturer": "MSR Productions",
      "deviceFirmwareVersion": "v0.1",
      "ignoreHttpsCertErrors": false,
      "sharedPolling": true,
      "sharedPollingId": "ambientweather-devicelab",
      "urlStatus": "https://example.local/weather.json",
      "paramNameCurrentAmbientLightLevel": "lastData.solarradiation <= 0 ? 0.0001 : lastData.solarradiation",
      "updateInterval": 120000,
      "sharedPollingInterval": 120000
    },
    {
      "deviceType": "AirQualitySensor",
      "enableLogging": true,
      "deviceID": "devicelab-ambient-pm25",
      "deviceName": "Device Lab Ambient PM2.5",
      "deviceManufacturer": "MSR Productions",
      "deviceModel": "v0.1",
      "deviceFirmwareVersion": "v0.1",
      "ignoreHttpsCertErrors": false,
      "sharedPolling": true,
      "sharedPollingId": "ambientweather-devicelab",
      "urlStatus": "https://example.local/weather.json",
      "paramNameAirQuality": "($aqi := $number(lastData.aqi_pm25); $aqi <= 50 ? 1 : $aqi <= 100 ? 2 : $aqi <= 150 ? 3 : $aqi <= 200 ? 4 : 5)",
      "paramNamePM2_5Density": "lastData.pm25_24h",
      "updateInterval": 120000,
      "sharedPollingInterval": 120000
    }
  ]
}
```

## JSONata Mapping

Simple dot notation still works because it is valid JSONata:

```json
"paramNamePM2_5Density": "lastData.pm25_24h"
```

Expressions can also transform values:

```json
"temperatureName": "sensor.temperatureC * 1.8 + 32"
```

JSONata functions are available:

```json
"humidityName": "$round(sensor.humidity, 1)"
```

## Supported Device Types

- `Switch`
- `Outlet`
- `LightBulb`
- `Sensor`
- `Fan`
- `MotionSensor`
- `ContactSensor`
- `LightSensor`
- `AirQualitySensor`
- `CarbonDioxideSensor`
- `SmokeSensor`
- `OccupancySensor`
- `DoorOpener`
- `GarageDoorOpener`
- `Window`
- `WindowCovering`
- `Valve`
- `Battery`

## Local Development

```bash
git clone https://github.com/martinshreder/homebridge-device-lab.git
cd homebridge-device-lab
npm install
npm run build
npm run lint
npm pack
```

Install a local package for testing:

```bash
sudo npm install -g ./homebridge-device-lab-*.tgz
sudo hb-service restart
```

If Homebridge runs under a service account with a user-local npm prefix, install into that account instead:

```bash
sudo -H -u sa_homebridge npm install -g /tmp/homebridge-device-lab-*.tgz
sudo hb-service restart
```

## Roadmap

- Random value generation (RNG) for synthetic test devices.
- More virtual accessory types that do not require external HTTP or MQTT sources.
- Cleaner config schema grouped around data sources, transforms, and HomeKit characteristics.
- Migration helpers for replacing dummy switches, random-number switch plugins, and simple MQTT mapping plugins.
