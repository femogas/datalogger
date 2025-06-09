# Global Configuration

This document describes the global configuration settings and functions available in the `global` package.

| Environment Variable | CLI Flag          | Type     | Description                                                      | Default       |
| -------------------- | ----------------- | -------- | ---------------------------------------------------------------- | ------------- |
| `LOG_LEVEL`          | `log-level`       | `string` | Sets the logging level (e.g., "debug", "info", "warn", "error"). | "info"        |
| `CONFIG_FILE`        | `config-file`     | `string` | Specifies the path to the configuration file.                    | "config.json" |
| `MAX_LOG_STORAGE`    | `max-log-storage` | `int64`  | Defines the maximum number of storage logs per variable.         | 1000          |

---

## `func InitializeGlobalConfiguration() *GlobalConfiguration`

Initializes and returns a new `GlobalConfiguration`. It reads environment variables and command-line flags to set the configuration values.

- **Returns:** A pointer to the initialized `GlobalConfiguration`.

## `func InitializeLogger(logLevel string) *logrus.Logger`

Initializes and returns a new `logrus.Logger` based on the specified log level.

- **Parameters:**
  - `logLevel`: The log level for the logger (e.g., "debug", "info", "warn").
- **Returns:** A pointer to the initialized `logrus.Logger`.

## Utility Functions

The package also provides several utility functions for handling default values and environment variables:

- `DefaultValueString(value interface{}, defaultValue string) string`
- `DefaultValueInt(value interface{}, defaultValue int) int`
- `DefaultValueInt64(value interface{}, defaultValue int64) int64`
- `GetEnv(key, defaultValue string) string`
- `GetEnvAsInt(name string, defaultValue int) int`
- `GetEnvAsInt64(name string, defaultValue int64) int64`
