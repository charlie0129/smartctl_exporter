# smartctl_exporter

Read the upstream README [prometheus-community/smartctl_exporter](https://github.com/prometheus-community/smartctl_exporter) .

## How is this fork different?

- Runs on newer Macs (and all supported platforms by upstream). The upstream will not run on newer Macs.
- Disables the ability to automatically discover devices and instead requires you to specify the device you want to monitor.
- Uses instant polling (reads SMART data when Prometheus is scraping). The upstream collects SMART data every some interval and caches the data, which is not ideal for Prometheus since the data can be stale and redundant.
- The upstream project uses `/usr/sbin/smartctl` binary by default (which is not always the case). This fork automatically finds the `smartctl` binary in your PATH (or you can specify the path to the binary using the `--smartctl.path` flag).

However, it is not a drop-in replacement. It differs in the following ways:

- You have to specify the device you want to monitor because the automatic device discovery is disabled.
- You need to reduce the scrape interval of Prometheus to a reasonable value (e.g., several minutes) because the exporter reads the SMART data when Prometheus is scraping. Reading SMART data in a short interval is meaningless and can cause unnecessary load on the system.

Read the [Usage](#usage) section for more information.

## Building

Make sure you have Golang installed. Then run:

```sh
CGO_ENABLED=0 go build -o smartctl_exporter .
```

Then you can move the `smartctl_exporter` binary to wherever you want. For example, you can move it to `/usr/local/bin`:

```sh
sudo mv smartctl_exporter /usr/local/bin
```

## Usage

First, you can check out the help message `smartctl_exporter --help`. There is one flag you need to care about:

- `--smartctl.device`: The device you want to monitor. For example, `/dev/disk0`. You can specify this flag multiple times to monitor multiple devices.

For example, to monitor `/dev/disk0` and `/dev/disk1`, you can run:

```sh
smartctl_exporter --smartctl.device=/dev/disk0 --smartctl.device=/dev/disk1
```

Then you can visit `http://localhost:9633/metrics` to see the metrics and configure Prometheus to scrape this endpoint (be sure to reduce the scrape interval of Prometheus to a reasonable value).

```
  - job_name: "smartctl_exporter"
    scrape_interval: 5m
    static_configs:
    - targets: ["127.0.0.1:9633"]
```
