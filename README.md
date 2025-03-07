# Comput3.ai Node Launcher

A utility for launching and monitoring Comput3.ai compute nodes.

## Overview

This tool helps you launch and maintain Comput3.ai compute nodes with automatic health monitoring and node replacement. It's designed to ensure your compute resources stay running even if individual nodes experience failures.

## Features

- **Multi-node launch**: Launch multiple compute nodes with a single command
- **Health monitoring**: Continuously check node health in the background
- **Auto-replacement**: Automatically replace failed nodes
- **Node balancing**: Alternates between fast and large node types
- **Docker support**: Run in a containerized environment for easy deployment

## Quick Start

Pull the container image:

```bash
docker pull ghcr.io/comput3ai/c3-launcher:latest
```

Run with environment variables directly:

```bash
docker run -e C3_API_KEY=your_api_key_here ghcr.io/comput3ai/c3-launcher:latest
```

Or use an environment file (recommended for security):

```bash
# Create a .env file with your credentials
echo "C3_API_KEY=your_api_key_here" > .env

# Run with the environment file
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest
```

Launch multiple nodes:

```bash
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 3
```

Run with custom health check polling interval:

```bash
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 2 --poll 60
```

Run as a background service:

```bash
docker run -d --name c3-launcher --restart unless-stopped --env-file ./.env \
  ghcr.io/comput3ai/c3-launcher:latest --nodes 4 --keep-running
```

Run with specific node type:

```bash
# Launch all nodes with large type
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 2 --type large
```

```bash
# Launch all nodes with fast type
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 2 --type fast
```

Run with auto-cleanup on exit:

```bash
# Launch nodes and stop them when the script exits (Ctrl+C)
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 2 --stop-on-exit
```

## Docker Usage

### Environment Variables

You can pass environment variables to the Docker container using the `-e` flag or an environment file:

```bash
# Using -e flag for each variable
docker run -e C3_API_KEY=your_api_key_here -e WORKLOAD_POLL=15 ghcr.io/comput3ai/c3-launcher:latest
```

```bash
# Using an environment file
# First create a .env file with your variables:
# C3_API_KEY=your_api_key_here
# WORKLOAD_POLL=15

docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest
```

### Command Line Arguments

Pass command line arguments after the image name:

```bash
# Launch 3 nodes
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 3
```

```bash
# Launch nodes with custom poll interval
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --nodes 2 --poll 60
```

```bash
# Launch with keep-running flag
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --keep-running
```

## Available Options

### Command Line Options

| Option | Description | Default |
|--------|-------------|---------|
| `--nodes` | Number of nodes to launch | 1 |
| `--keep-running` | Keep nodes running with hourly renewal | False |
| `--poll` | Node health check interval in seconds | 30 |
| `--type` | Node type to launch (fast, large, or alternate) | alternate |
| `--stop-on-exit` | Stop all workloads when the script exits | False |

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `C3_API_KEY` | Your Comput3.ai API key | (Required) |
| `WORKLOAD_POLL` | Node health check interval in seconds | 30 |

## Node Monitoring

The launcher includes parallel, per-node health monitoring:

- Each node gets its own dedicated monitoring thread
- Adds a 5-second delay after launching each node to allow for initialization
- Periodically checks each node's health at configurable intervals
- Uses a "3 strikes" approach: initial check + 2 retries before considering a node dead
- Automatically stops the failed node
- Launches a replacement node of the same type
- Provides real-time status updates in the console

Monitoring works as follows:
1. Initial health check right after the 5-second boot delay
2. Regular polling based on the configured interval (default: 30 seconds)
3. If a node fails a check, it gets 2 more chances (consecutive failures)
4. After 3 consecutive failures, the node is considered dead and replaced
5. Any successful check resets the failure counter

## Node Types

The launcher can work with different node types:

- **ollama_webui:fast**: Optimized for quick response times
- **ollama_webui:large**: Optimized for larger workloads

You can specify which type to use with the `--type` parameter:

```bash
# Launch only large nodes
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --type large
```

```bash
# Launch only fast nodes
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --type fast
```

```bash
# Alternate between fast and large (default behavior)
docker run --env-file ./.env ghcr.io/comput3ai/c3-launcher:latest --type alternate
```

## Troubleshooting

Common issues:

- **Authentication errors**: Check that your C3_API_KEY is valid
- **Network errors**: Ensure you have internet connectivity
- **Container stops**: Check Docker logs with `docker logs c3-launcher`

## License

[MIT License](LICENSE)
