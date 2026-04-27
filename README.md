

<a href="https://hauke.cloud" target="_blank"><img src="https://img.shields.io/badge/home-hauke.cloud-brightgreen" alt="hauke.cloud" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud" target="_blank"><img src="https://img.shields.io/badge/github-hauke.cloud-blue" alt="hauke.cloud Github Organisation" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud/readme-management" target="_blank"><img src="https://img.shields.io/badge/template-default-orange" alt="Repository type - default" style="display: block;" /></a>


# database-iot-gorm


<img src="https://raw.githubusercontent.com/hauke-cloud/.github/main/resources/img/organisation-logo-small.png" alt="hauke.cloud logo" width="109" height="123" align="right">


A Go package providing GORM models and database migrations for IoT sensor data storage. This package contains reusable database models for various sensor types including moisture sensors, valves, water level sensors, and room sensors.

## Features

- **GORM Models**: Pre-defined models for:
  - Devices (base model for all sensor types)
  - Battery status
  - Link quality
  - Moisture measurements
  - Valve measurements
  - Water level measurements
  - Room measurements

- **Database Migrations**: SQL migrations for PostgreSQL using golang-migrate
  - Common tables (devices, batteries, link_qualities)
  - Sensor-specific tables (moisture_measurements, valve_measurements, water_level_measurements, room_measurements)

- **Selective Migration**: Run migrations only for the sensor types you need


## 🚀 Getting started

### Installation

```bash
go get github.com/hauke-cloud/database-iot-gorm
```

### Import the package

```go
import databaseiotgorm "github.com/hauke-cloud/database-iot-gorm"
```

### Run migrations

```go
import (
    "gorm.io/gorm"
    "go.uber.org/zap"
    databaseiotgorm "github.com/hauke-cloud/database-iot-gorm"
)

// Initialize your GORM database connection
var db *gorm.DB
var logger *zap.Logger

// Run migrations for specific sensor types
sensorTypes := []string{"moisture", "valve", "water_level", "room"}
err := databaseiotgorm.RunMigrationsForSensorTypes(db, sensorTypes, logger)
if err != nil {
    // Handle error
}
```

### Use the models

```go
// Create a device
device := databaseiotgorm.Device{
    DeviceID:   "moisture-sensor-01",
    DeviceName: "Garden Moisture Sensor",
    SensorType: "moisture",
    ShortAddr:  "0xBF16",
    IEEEAddr:   "0x00124b001234abcd",
}
db.Create(&device)

// Create a measurement
measurement := databaseiotgorm.MoistureMeasurement{
    DeviceID:    device.ID,
    Timestamp:   time.Now(),
    Temperature: &temperature,
    Humidity:    &humidity,
}
db.Create(&measurement)
```

## Available Models

- `Device` - Base device information
- `Battery` - Battery status tracking
- `LinkQuality` - Zigbee link quality tracking
- `MoistureMeasurement` - Soil moisture and temperature
- `ValveMeasurement` - Irrigation valve status and metrics
- `WaterLevelMeasurement` - Water level readings
- `RoomMeasurement` - Room temperature and humidity

## Migration Types

The package supports selective migration based on sensor types:
- `common` - Always runs (devices, batteries, link_qualities tables)
- `moisture` - Moisture sensor measurements
- `valve` - Valve sensor measurements
- `water_level` - Water level sensor measurements
- `room` - Room sensor measurements



## 📄 License

This Project is licensed under the GNU General Public License v3.0

- see the [LICENSE](LICENSE) file for details.


## :coffee: Contributing

To become a contributor, please check out the [CONTRIBUTING](CONTRIBUTING.md) file.


## :email: Contact

For any inquiries or support requests, please open an issue in this
repository or contact us at [contact@hauke.cloud](mailto:contact@hauke.cloud).

