[![Build Status](https://travis-ci.org/viasite-ansible/ansible-role-kapacitor.svg?branch=master)](https://travis-ci.org/viasite-ansible/ansible-role-kapacitor)

Kapacitor
=========

An Ansible role to install, configure, and manage [Kapacitor](https://github.com/influxdb/kapacitor) (an open source framework for processing, monitoring, and alerting on time series data).

Requirements
------------

Prior knowledge/experience with [InfluxDB](https://github.com/influxdb/influxdb) and Kapacitor is highly recommended. Full documentation is available [here](https://docs.influxdata.com/kapacitor/v0.2/introduction/getting_started/).

Installation
------------

Either clone this repository, or install through Ansible Galaxy directly using the command:

```
ansible-galaxy install rossmcdonald.kapacitor
```

Role Variables
--------------

The high-level variables are stored in the `defaults/main.yml` file. The most important ones being:

```
# Channel of Kapacitor to install (stable, unstable, nightly)
kapacitor_install_version: stable
```

More advanced configuration options are stored in the `vars/main.yml` file, which includes all of the necessary bells and whistles to tweak your configuration.

Task templates
--------------

Tasks should be defined with `kapacitor_tasks`, variables `kapacitor_tasks_to_enable`, `kapacitor_stream_thresholds`, `kapacitor_stream_thresholds_disable` has been deprecated and will be removed in a future release.

``` yaml
kapacitor_tasks:
  - name: alert_load_average
    template: tick/stream_threshold.tick.j2
    measurement: system
    field: load1
    groupBy: host
    period: 3m
    every: 1m
    warn: "> 4"
    crit: "> 8"

  - name: disk_used_percent
    template: tick/stream_threshold.tick.j2
    state: disabled
    measurement: disk
    field: used_percent
    groupBy: ['host', 'path']
    period: 1m
    every: 1m
    warn: "> 95"
    crit: "> 97"

  - name: stream_disk_util
    template: ../../vars/kapacitor/stream_disk_util.tick

```

Default task values:

``` yaml
type: stream
dbrp: "{{ kapacitor_dbrp }}"
state: enabled
```

Dependencies
------------

Access to an InfluxDB instance is required to use Kapacitor, though they do not need to be located on the same server (can be configured using the `kapacitor_influxdb_urls` variable). Apart from that, no other Ansible dependencies are required.

This role was tested and developed with Ansible 1.9.4.

Example Playbook
----------------

An example playbook is included in the `test.yml` file. There is also a `Vagrantfile`, which can be used for quick local testing leveraging [Vagrant](https://www.vagrantup.com/). Since Kapacitor's use-case is very user-specific, the main goal of this repo is to install, configure, and setup Kapacitor in a customizable way. Currently to enable TICKscripts, specify them as a dictionary in the `kapacitor_tasks_to_enable` variable like so:

```
      kapacitor_tasks_to_enable:
        - name: cpu_alert
          type: stream
          tick: cpu_alert.tick
          dbrp: telegraf.default
```

Where `tick` is the path to the TICKscript file. If placed in the `/files` directory of the role repository, you can just specify the name directly (for example, `cpu_alert.tick`).

Contributions and Feedback
--------------------------

Any contributions are welcome. For any bugs or feature requests, please open an issue through Github.

License
-------

MIT

Author
------

Created by [Ross McDonald](https://github.com/rossmcdonald).

