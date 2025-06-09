# Guide to Integrating the Datalogger Custom Connector

Welcome! This guide will help you integrate the Datalogger Custom Connector package into your project. We’ll provide an overview of the main features and guide you through the necessary steps to develop, configure, and integrate your connector.

## Introduction

This project provides a framework for creating connectors that interact with a Redis-based system, manage unique UUIDs for nodes or endpoints, and utilize Pub/Sub mechanisms for inter-process communication. This guide is intended for developers looking to extend the system's functionality by creating and integrating custom connectors.

## Prerequisites

- **Go** version 1.23.1 or higher.
- **Redis** installed and running.
- Familiarity with Go language and concepts like contexts, logging, and error handling.

## Installation

To use the Datalogger Custom Connector in your project, add it as a dependency in your `go.mod` file:

```bash
go get github.com/femogas/datalogger
```

Ensure that the package is properly installed and available in your project.

## Project Structure

The project is divided into several key packages:

- **application**: Contains the main application logic, including the `Connector` interface and signal handling.
- **application/configuration**: Manages global application configuration.
- **redis**: Contains functions to interact with Redis, manage the data stream, and handle Pub/Sub communication.
- **redis/configuration**: Manages Redis-specific configuration.
- **redis/pubsub**: Manages Pub/Sub communication through Redis.
- **redis/uuid**: Manages UUID mapping and persistence.
- **server**: Provides an HTTP server for the connector interface.
- **server/configuration**: Manages server-specific configuration.

## Global Configuration

To initialize global configuration, use the `InitializeGlobalConfiguration()` function from the `global` package. This function reads environment variables and command-line flags to set configuration values.

```go
import "github.com/femogas/datalogger/application/configuration"

config := global.InitializeGlobalConfiguration()
```

This configuration is essential for setting up the connector correctly and should be integrated at the beginning of your application.

## Logger Initialization

The logger is essential for monitoring the operation of your connector. Initialize it using `InitializeLogger(logLevel string)` from the `global` package, where `logLevel` is a string representing the desired logging level (e.g., "debug", "info", "warn", "error").

```go
import "github.com/femogas/datalogger/application/configuration"

logger := global.InitializeLogger(config.LogLevel)
```

The logger will help you track the connector's activities and debug issues effectively.

## Connecting to Redis

To interact with Redis, create a new client using `NewClient(logger *logrus.Logger, ctx context.Context)` from the `redis` package.

```go
import (
    "context"
    "github.com/femogas/datalogger/redis"
    "github.com/sirupsen/logrus"
)

ctx, cancel := context.WithCancel(context.Background())
defer cancel()

redisClient, err := redis.NewClient(logger, ctx)
if err != nil {
    logger.Fatalf("Error initializing Redis client: %v", err)
}
defer redisClient.Close()
```

Proper context management ensures the Redis client connection is closed when it is no longer needed.

## UUID Management

UUIDs are used to uniquely identify nodes or endpoints. The UUID management process requires an explicit call to `GenerateUUIDMap` to synchronize the UUIDs with Redis.

```go
// Generate UUID map
err = redisClient.GenerateUUIDMap(connector.CreateValidKeys)
if err != nil {
    logger.WithError(err).Fatal("Failed to generate UUID map")
}
```

This step is essential for ensuring each node is uniquely identified and can be tracked throughout the system.

## Connector Implementation

To create a custom connector, implement the `Connector` interface, which is defined in the `application/producer` package. This interface includes methods for setting up, starting, stopping, and checking the connector's status.

### Connector Interface

Below is a description of the `Connector` interface:

```go
type Connector interface {
    Setup(globalconfig *global.GlobalConfiguration) error
    Start() error
    Status() Status
    Stop() error
    CreateValidKeys() map[string]interface{}
}
```

You must implement these methods to define how your connector should behave during its lifecycle.

### Example Usage

To use the connector, you need to create an instance of your connector struct and initialize it with the necessary dependencies, such as Redis and global configuration. The following `main` function example demonstrates how to set up and run the application:

```go
package main

import (
	"context"
	"sync"

	producer "github.com/femogas/datalogger/application"
	global "github.com/femogas/datalogger/application/configuration"
	"github.com/femogas/datalogger/redis"
	"github.com/femogas/datalogger/server"
	"github.com/sirupsen/logrus"
)

// YourNewConnector manages client connections and configurations.
type YourNewConnector struct {
	mutex         sync.Mutex
	running       bool
	Logger        *logrus.Logger
	Configuration *global.Connector
	Redis         *redis.Client
}

// Setup initializes the connector.
func (c *YourNewConnector) Setup(globalconfig *global.GlobalConfiguration) error {
	// Add your setup logic here
	return nil
}

// Start begins the connector's operation.
func (c *YourNewConnector) Start() error {
	c.mutex.Lock()
	defer c.mutex.Unlock()
	c.running = true
	// Add your start logic here
	return nil
}

// Status returns the current status of the connector.
func (c *YourNewConnector) Status() producer.Status {
	// Add your status logic here
	return producer.Status{}
}

// Stop halts the connector's operation.
func (c *YourNewConnector) Stop() error {
	c.mutex.Lock()
	defer c.mutex.Unlock()
	c.running = false
	// Add your stop logic here
	return nil
}

// CreateValidKeys generates a map of valid keys.
func (c *YourNewConnector) CreateValidKeys() map[string]interface{} {
	validKeys := make(map[string]interface{})
	// Add your logic to create valid keys here
	return validKeys
}

func main() {
	globalConfig := global.InitializeGlobalConfiguration()
	logger := global.InitializeLogger(globalConfig.LogLevel)

	ctx, cancel := context.WithCancel(context.Background())
	defer cancel()

	redisClient, err := redis.NewClient(logger, ctx)
	if err != nil {
		logger.WithError(err).Fatal("Failed to initialize Redis client")
	}
	defer redisClient.Close()

	connector := &YourNewConnector{
		Redis:  redisClient,
		Logger: logger,
	}

	if err := connector.Setup(globalConfig); err != nil {
		logger.Fatalf("Error setting up connector: %v", err)
	}

	err = redisClient.GenerateUUIDMap(connector.CreateValidKeys)
	if err != nil {
		logger.WithError(err).Fatal("Failed to generate UUID map")
	}

	appInstance := &producer.Main{
		Context:   ctx,
		Cancel:    cancel,
		Connector: connector,
		Logger:    logger,
		Redis:     redisClient,
	}

	go redisClient.ListenForCommands(connector.Start, connector.Stop)

	httpServer := server.NewServer(appInstance, logger)
	if err := httpServer.Start(); err != nil {
		logger.Fatalf("Error starting server: %v", err)
	}

	producer.SetupSignalHandling(appInstance, connector)

	<-ctx.Done()
}
```

This example demonstrates how to integrate the connector lifecycle with Redis and HTTP server functionalities.

## Signal Handling

To ensure a clean application shutdown, use `SetupSignalHandling` from the `producer` package. This will listen for system signals like `SIGINT` and `SIGTERM` and call appropriate methods to stop the connector.

```go
import "github.com/femogas/datalogger/application"

producer.SetupSignalHandling(appInstance, connector)
```

Proper signal handling ensures that the application resources are released gracefully.

## Running the Server

The Datalogger Custom Connector includes an HTTP server for integration purposes. Use `NewServer` from the `server` package to initialize and run the HTTP server:

```go
import "github.com/femogas/datalogger/server"

httpServer := server.NewServer(appInstance, logger)
if err := httpServer.Start(); err != nil {
    logger.Fatalf("Error starting server: %v", err)
}
```

The server provides a simple interface to monitor and control the connector, facilitating integration with other components.

### Server Endpoints

The HTTP server exposes the following endpoints:

- **`/healthz`**: A health check endpoint that returns `Health ok!` with a status code of `200 OK`.
- **`/status`**: Returns the current status of the connector in JSON format.
- **`/uuid-mapping`**: Returns the current UUID mapping from Redis in JSON format.

## Pub/Sub Listener

The Redis client includes a listener for commands that control the connector. Use the `ListenForCommands` method to start or stop the connector based on messages received on the `remote-control` channel.

```go
go redisClient.ListenForCommands(connector.Start, connector.Stop)
```

This setup allows you to control the connector dynamically, responding to external requests for starting or stopping operations.

## Versioning

We use [SemVer](https://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/femogas/datalogger/tags).

## Authors

- **Paolo Fabris** - _Initial work_ - [ubyte.it](https://ubyte.it/)

See also the list of [contributors](https://github.com/femogas/datalogger/blob/main/CONTRIBUTORS.md) who participated in this project.

## License

This project is licensed under the MIT License. See the [LICENSE](https://github.com/femogas/datalogger/blob/main/LICENSE) file for details.
