<!-- llm-readme-management spec=1 commit=7dd046b5b9359cecc3960209f3aa3a804167e3a9 template=golang model=qwen3.8-27b-q4 digest=b3d4b07f2e19 generated=2026-09-08T13:30:30Z -->
<a href="https://hauke.cloud" target="_blank"><img src="https://img.shields.io/badge/home-hauke.cloud-brightgreen" alt="hauke.cloud" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud" target="_blank"><img src="https://img.shields.io/badge/github-hauke.cloud-blue" alt="hauke.cloud Github Organisation" style="display: block;" /></a>
<a href="https://github.com/hauke-cloud/llm-readme-management" target="_blank"><img src="https://img.shields.io/badge/template-golang-orange" alt="Repository type - golang" style="display: block;" /></a>


# Database Iot Gorm


<img src="https://raw.githubusercontent.com/hauke-cloud/.github/main/resources/img/organisation-logo-small.png" alt="hauke.cloud logo" width="109" height="123" align="right">


<llm header hint="Name the Go module path and say whether this is a service, a CLI or a library.">

This Go library (`github.com/hauke-cloud/database-iot-gorm`) provides GORM models and embedded golang-migrate SQL migrations for storing IoT sensor data—moisture, valve, water level, and room readings—in PostgreSQL. It is consumed by services such as the mqtt-sensor-exporter, which calls its migration function against databases selected by Kubernetes Custom Resources.

</llm>


## :book: Description

<llm description>

`database-iot-gorm` is a Go library that provides GORM models and embedded golang-migrate SQL migrations for storing IoT sensor data in PostgreSQL. It defines a normalized schema—shared `devices`, `batteries`, and `link_qualities` tables plus per-sensor-type measurement tables—and exposes a single entry point, `RunMigrationsForSensorTypes`, that applies only the migration sets you request.

The package is a library, not an application: no `main`, no CLI, no Dockerfile. You import it, pass an open `*gorm.DB` connection, a slice of sensor-type names, and a `*zap.Logger`, and it handles the rest.

- GORM structs for `Device`, `Battery`, `LinkQuality`, and four measurement types, all foreign-keyed to `devices(id)` with cascade delete.
- Per-sensor-type migration sets (common, moisture, valve, water_level, room) embedded via `//go:embed` and applied through golang-migrate's `iofs` driver.
- Automatic recovery from dirty migration states and post-migration verification that expected tables exist in the `public` schema.
- Historical normalization migrations that convert a legacy denormalized schema to the current one while preserving data.

Within the `hauke-cloud` organisation the library is consumed by the `mqtt-sensor-exporter` service, which watches `databases.iot.hauke.cloud` Kubernetes Custom Resources and calls the migration function against the database selected by each resource's `supportedSensorTypes` field.

</llm>


## :clipboard: Requirements

<llm requirements hint="Give the Go version from the go directive in go.mod. Mention Docker only if the repository actually builds an image.">

- Go 1.26.2 toolchain (pinned in `go.mod`).
- A reachable PostgreSQL database. The connecting user must hold `CREATE TABLE` and `CREATE INDEX` privileges in the `public` schema.
- A GORM PostgreSQL driver (for example `gorm.io/driver/postgres`) in your consuming module. This package does not bundle one.
- pre-commit, required to install and run the repository's git hooks before contributing.

</llm>


## 🚀 Getting started

<llm getting_started hint="Cover go build, go run and go test with the real package paths. If a Makefile or Taskfile exists, prefer its targets over raw go commands.">

1. Clone the repository and enter the working directory.

```bash
git clone https://github.com/hauke-cloud/database-iot-gorm.git
cd database-iot-gorm
```

2. Verify the module compiles. This is the conventional Go build command; the repository ships no Makefile or Taskfile.

```bash
go build ./...
```

3. Run the test suite. This is the conventional Go test command; no `*_test.go` files exist yet, so the step confirms the package loads without error.

```bash
go test ./...
```

4. Install the pre-commit hooks that `CONTRIBUTING.md` requires before you push.

```bash
pre-commit install
```

At this point the library is ready to import. In a consuming service you would add the module with `go get github.com/hauke-cloud/database-iot-gorm`, open a GORM PostgreSQL connection, and call `RunMigrationsForSensorTypes(db, sensorTypes, logger)` to create the schema for the sensor types you need.

</llm>


## :airplane: Usage

<llm usage hint="For a library, show a small import-and-call example using real exported identifiers. For a service or CLI, show how it is started and the flags or subcommands it accepts.">

This package is a Go library. You import it into your service, open a GORM PostgreSQL connection, and call the exported functions and types.

**Run migrations for selected sensor types**

The primary entry point is `RunMigrationsForSensorTypes`. It applies the `common` migration set first, then only the sensor-type sets you request; unknown types are logged and skipped.

```go
import (
	"github.com/hauke-cloud/database-iot-gorm"
	"go.uber.org/zap"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
)

db, err := gorm.Open(
	postgres.Open("host=localhost user=postgres dbname=iot sslmode=disable"),
	&gorm.Config{},
)
if err != nil {
	log.Fatal(err)
}

logger, _ := zap.NewProduction()

err = databaseiotgorm.RunMigrationsForSensorTypes(
	db,
	[]string{"moisture", "valve"},
	logger,
)
if err != nil {
	log.Fatal(err)
}
```

**Query sensor data with the GORM models**

Seven exported structs map to the normalized schema: `Device`, `Battery`, `LinkQuality`, `MoistureMeasurement`, `ValveMeasurement`, `WaterLevelMeasurement`, and `RoomMeasurement`. Use them directly in GORM queries against the same `*gorm.DB` handle.

```go
var readings []databaseiotgorm.MoistureMeasurement
if err := db.Find(&readings).Error; err != nil {
	log.Fatal(err)
}
```

The database connection string, credentials, and schema are entirely your responsibility; this package does not read environment variables or configuration files. The database user needs `CREATE TABLE` and `CREATE INDEX` privileges in the `public` schema for migrations to succeed.

</llm>


## :wrench: Configuration

<llm configuration hint="Environment variables and CLI flags, taken from the flag definitions or the config struct.">

This repository exposes no environment variables, CLI flags, or configuration files. It is a library, and its only configuration surface is the parameter list of the single exported function `RunMigrationsForSensorTypes` in `migrations.go`:

- **`db *gorm.DB`** (required) — An open GORM connection to a PostgreSQL database. The caller is responsible for constructing the connection (host, DSN, credentials) and supplying a PostgreSQL driver such as `gorm.io/driver/postgres`.
- **`sensorTypes []string`** (required) — Which per-sensor migration sets to apply. Valid values are `"moisture"`, `"valve"`, `"water_level"`, and `"room"`. The `common` set (shared `devices`, `batteries`, `link_qualities` tables) always runs first. Any value outside the four listed is logged as a warning and skipped. In the intended deployment this slice is populated from the `spec.supportedSensorTypes` field of a `databases.iot.hauke.cloud` Kubernetes Custom Resource.
- **`logger *zap.Logger`** (required) — A `zap` logger used for migration progress messages, dirty-state warnings, and post-migration table-verification results.

There is no configuration file, no `flag` package usage, and no `os.Getenv` call anywhere in the module. All runtime behaviour is determined by the three arguments above and the state of the target database. The full set of inputs is therefore complete as listed; there is no long tail to point to.

</llm>


## :hammer: Development

<llm development hint="Include go test, go vet and gofmt only where the CI workflows actually run them.">

Before pushing, install the pre-commit hooks and run them over every file:

```bash
pre-commit install
pre-commit run --all-files
```

The configured hooks (pre-commit-hooks v4.4.0, gitleaks v8.18.0) handle code-style checks and secret detection. Run `pre-commit autoupdate` to keep hook versions current.

**PR title format.** The `pr-title.yml` workflow rejects pull requests whose title does not follow a conventional-commit pattern. The type must be one of `fix`, `feat`, `docs`, `ci`, or `chore`; the subject after the colon must start with an uppercase letter. A `[WIP]` prefix is permitted.

**No automated build or test gate.** The repository contains no `*_test.go` files, and the three CI workflows (`lock.yml`, `pr-title.yml`, `stale-actions.yml`) do not run `go test`, `go vet`, `gofmt`, or any build step. You can verify compilation locally:

```bash
go build ./...
```

No generated files require regeneration. The embedded SQL migrations under `migrations/` are plain files with no code-generation step.

</llm>


## 📄 License

This Project is licensed under the GNU General Public License v3.0

- see the [LICENSE](LICENSE) file for details.


## :coffee: Contributing

To become a contributor, please check out the [CONTRIBUTING](CONTRIBUTING.md) file.


## :email: Contact

For any inquiries or support requests, please open an issue in this
repository or contact us at [contact@hauke.cloud](mailto:contact@hauke.cloud).
