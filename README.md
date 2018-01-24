# Ansible Role: `nftables`

[![Build Status](https://travis-ci.org/Frzk/ansible-role-nftables.svg?branch=master)](https://travis-ci.org/Frzk/ansible-role-nftables)

This Ansible role allows you to install `nftables` and manage its configuration.

For more information about `nftables`, please check [the official project page](http://netfilter.org/projects/nftables/index.html).


## Role variables

Variables and *properties* in bold are mandatory. Others are optional.

| Variable name            | Description                                        | Default value        |
| ------------------------ | -------------------------------------------------- | -------------------- |
| `nftables_config_file`   | Path to the configuration file.                    | `/etc/nftables.conf` |
| `nftables_tables`        | A list of [table](#table-properties).              | `[]`                 |


### table properties

| Property name  | Description                                                                                                 | Default value |
| -------------- | ----------------------------------------------------------------------------------------------------------- | ------------- |
| **`name`**     | Name of the table.                                                                                          |               |
| `family`       | Address family of the table. If specified, must be either `ip`, `ip6`, `inet`, `arp`, `bridge` or `netdev`. | `ip`          |
| `sets`         | A list of [set](#set-properties).                                                                           |               |
| `maps`         | A list of [map](#map-properties).                                                                           |               |
| `verdict_maps` | A list of [verdict_map](#verdict_map-properties).                                                           |               |
| `chains`       | A list of [chain](#chain-properties).                                                                       |               |

[Documentation](https://manpages.debian.org/testing/nftables/nftables.8.en.html#TABLES)


### set properties

| Property name   | Description                                                                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`name`**      | Name of the set.                                                                                                                                  |
| **`type`**      | Address type of the elements contained in the set. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto` or `mark`. |
| `size`          | Number of elements the set can contain.                                                                                                           |
| `policy`        | The set selection policy. If specified, must be either `performance` or `memory`.                                                                 |
| `timeout`       | How long the elements stay in the set.                                                                                                            |
| `flags`         | A list of flags. If specified, must contain at least one of the following : `constant`, `interval`, `timeout`.                                    |
| `gc_interval`   | Garbage collection interval.                                                                                                                      |
| `elements`      | A list of elements contained in the set. Elements must conform to the set `type`.                                                                 |

[Documentation](https://manpages.debian.org/nftables/nftables.8.en.html#SETS)


### map properties

| Property name     | Description                                                                                                                             |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **`name`**        | Name of the map.                                                                                                                        |
| **`keys_type`**   | Type of the keys. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto` or `mark`.                        |
| **`values_type`** | Type of the values. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto`, `mark`, `counter` or `quota`.  |
| `elements`        | A list of [elements](#map-element-properties) contained in the map. Elements must conform to the map `keys_type` and `values_type`.     |

[Documentation](https://manpages.debian.org/nftables/nftables.8.en.html#MAPS)

#### map element properties

| Property name   | Description                    |
| --------------- | ------------------------------ |
| **`key`**       | Key value.                     |
| **`value`**     | Value associated with the key. |


### verdict_map properties

A `verdict_map` is just a special case of `map` where the `values_type` is always `verdict`. As such, there is no `values_type` property. Also, elements contained in a `verdict_map` have a `verdict` property instead of the `value` property.

| Property name     | Description                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| **`name`**        | Name of the map.                                                                                                 |
| **`keys_type`**   | Type of the keys. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto` or `mark`. |
| `elements`        | A list of [elements](#verdict_map-element-properties) contained in the verdict map.                              |

#### verdict_map element properties

| Property name   | Description                      |
| --------------- | -------------------------------- |
| **`key`**       | Key value.                       |
| **`verdict`**   | Verdict associated with the key. |


### chain properties

| Property name | Description                                               |
| ------------- | --------------------------------------------------------- |
| **`name`**    | Name of the chain.                                        |
| **`base`**    | [Base rule](#base-properties) for the chain.              |
| `rules`       | List of [rules](#rule-properties) contained in the chain. |

[Documentation](https://manpages.debian.org/nftables/nftables.8.en.html#CHAINS)

#### base properties

| Property name  | Description                                                                    |
| -------------- | ------------------------------------------------------------------------------ |
| **`type`**     | The type of the chain. Must be either `filter`, `nat` or `route`.              |
| **`hook`**     | Hook where the chain is attached. Available values depend on `type`.           |
| **`priority`** | Integer determining the order of the chains attached to the same `hook`.       |
| `policy`       | Default policy for the chain. If specified, must be either `accept` or `drop`. |

[Documentation](https://manpages.debian.org/nftables/nftables.8.en.html#CHAINS)

#### rule properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#RULES)

| Property name   | Description                                              |
| --------------- | -------------------------------------------------------- |
| **`position`**  | Integer determining the order of the rules in the chain. |
| **`statement`** | Rule statement.                                          |
| `comment`       | A comment describing the rule.                           |


## Example

Here is a small example of what your file should look like.

**IMPORTANT**: DO NOT use this as your firewall !

```yaml
---
nftables_flush_ruleset: yes
nftables_config_path: /etc/nftables.rules
nftables_tables:
  - name: firewall
    family: inet

    sets:
      - name: "set1"
        type: 
        size: 10
        policy: "performance"
        timeout: "1d"
        flags:
          - "timeout"
          - "interval"
        gc_interval: "12h" 
        elements:
          - 192.0.2.1
          - 192.0.2.2

    maps:
      - name: "map1"
        keys_type: "inet_service"
        values_type: "ipv4_addr"
        elements:
          - key: ssh
            value: "192.0.2.10"
      - name: "map2"
        keys_type: "inet_service"
        values_type: "ipv4_addr"
        elements:
          - key: ftp
            value: "192.0.2.25"

    verdict_maps:
      - name: "vmap1"
        keys_type: "inet_service"
        elements:
          - key: "192.0.2.10"
            value: "accept"

    chains:
      - name: "My input filter"
        base:
          type: "filter"
          hook: "input"
          priority: 0
          policy: "drop"
        rules:
          - position: 2
            statement: "ct state invalid log prefix 'Invalid_IN: ' drop"
            comment: "Log and drop invalid packets.
          - position: 1
            statement: "iif lo accept"
          - position: 3
            statement: "ct state {established,related} accept"

      - name: "My output filter"
        base:
          type: "filter"
          hook: "output"
          priority: -10
          policy: "accept"
        rules:
          - position: 1
            statement: "ip daddr 192.0.2.100 counter"
...
```


## Contributing


