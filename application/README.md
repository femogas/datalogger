# Application

This document describes the main components and interfaces of the `application` package, which is responsible for the main application logic, including signal handling and connector management.

---

## `func SetupSignalHandling(application *Main, connector Connector)`

Sets up signal handling for graceful shutdown of the application. Listens for `SIGINT` and `SIGTERM` signals and initiates shutdown procedures.

- **Parameters:**
  - `application`: A pointer to the `Main` application structure.
  - `connector`: The connector to be managed during shutdown.

## `func ParseHostFromEndpointURL(endpointURL string) (string, error)`

Parses an endpoint URL and extracts the hostname.

- **Parameters:**

  - `endpointURL`: A string representing the URL endpoint (e.g., `"https://example.com:443"`).

- **Returns:**
  - `string`: The extracted hostname (e.g., `example.com`).
  - `error`: An error if the URL cannot be parsed.

## `type Connector interface`

Defines the methods for setting up, starting, stopping, and checking the status of a connector.

- **Methods:**
  - `Setup(globalconfig *global.GlobalConfiguration) error`: Sets up the connector with the given global configuration.
  - `Start() error`: Starts the connector.
  - `Status() Status`: Retrieves the status of the connector.
  - `Stop() error`: Stops the connector.
  - `CreateValidKeys() map[string]interface{}`: Creates a map of valid keys from the given configurations.

## `type Status`

Represents the overall status of the connector, including the status of each endpoint.

- **Fields:**
  - `Connector []StatusEndpoint`: A list of endpoint statuses.

## `type StatusEndpoint`

Represents the status of an individual endpoint.

- **Fields:**
  - `Endpoint string`: The endpoint identifier.
  - `Status bool`: The status of the endpoint.

## `type Main`

Contains the main components of the application, including the context, connectors, logger, and Redis client.

- **Fields:**
  - `Context context.Context`: The application context.
  - `Cancel context.CancelFunc`: The function to cancel the application context.
  - `Connector Connector`: The connector instance.
  - `Logger *logrus.Logger`: The logger for logging application events.
  - `Redis *redis.Client`: The Redis client for data handling.

# Certificate

The `certificate` package provides functionality for generating and handling self-signed X.509 certificates for both RSA and ECDSA keys.

## `func GenerateCertificate(host string, rsaBits int, validFor time.Duration) (certPEM, keyPEM []byte, err error)`

Creates a self-signed X.509 certificate using an RSA key.

- **Parameters:**

  - `host`: A comma-separated list of hostnames and IP addresses.
  - `rsaBits`: The size of the RSA key.
  - `validFor`: The duration for which the certificate is valid.

- **Returns:**
  - `certPEM`: The certificate in PEM format.
  - `keyPEM`: The private key in PEM format.
  - `err`: An error if generation fails.

## `func GenerateECDSACertificate(host string, curve elliptic.Curve, validFor time.Duration) (certPEM, keyPEM []byte, err error)`

Creates a self-signed X.509 certificate using an ECDSA key.

- **Parameters:**

  - `host`: A comma-separated list of hostnames and IP addresses.
  - `curve`: The elliptic curve to use.
  - `validFor`: The duration for which the certificate is valid.

- **Returns:**
  - `certPEM`: The certificate in PEM format.
  - `keyPEM`: The private key in PEM format.
  - `err`: An error if generation fails.
