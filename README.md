# Ansible Role: `nftables`

This Ansible role allows you to install `nftables` and manage its configuration.

For more information about `nftables`, please check [the official project page](http://netfilter.org/projects/nftables/index.html).


## Role variables

Variables and *properties* in bold are mandatory. Others are optional.


| Variable name            | Description                                                   | Default value      |
| ------------------------ | ------------------------------------------------------------- | ------------------ |
| `nftables_flush_ruleset` | Wether we should flush the current ruleset or not.            | yes                |
| `nftables_config_file`   | Path to the configuration file.                               | /etc/nftables.conf |
| `nftables_tables`        | A list of `table` *objects* representing the nftables tables. |                    |

Example:

```yaml
---
nftables_flush_ruleset: yes
nftables_config_file: /etc/firewall.rules
nftables_tables:
- name: firewall
  family: inet
- name: nat
  family: inet
...
```


### `table` properties

[DOC](https://manpages.debian.org/testing/nftables/nftables.8.en.html#TABLES)

| Property name  | Description                                                                                                 | Default value |
| -------------- | ----------------------------------------------------------------------------------------------------------- | ------------- |
| **`name`**     | Name of the table.                                                                                          |               |
| `family`       | Address family of the table. If specified, must be either `ip`, `ip6`, `inet`, `arp`, `bridge` or `netdev`. | `ip`          |
| `sets`         | A list of `set` *objects*.                                                                                  |               |
| `maps`         | A list of `map` *objects*.                                                                                  |               |
| `verdict_maps` | A list of `verdict_map` *objects*.                                                                          |               |
| `chains`       | A list of `chain` *objects*.                                                                                |               |

**Example:**

```yaml
---
nftables_tables:
- name: firewall_ipv4
  family: ip
  sets:
    [...]
  chains:
    [...]
...
```


### `set` properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#SETS)

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

**Example:**

```yaml
---
nftables_tables:
[...]
    sets:
      - name: blacklist_ipv4
        family: ipv4_addr
        policy: performance
        timeout: 24h
        elements:
          - 192.168.0.45
          - 192.168.0.46
      - name: balcklist_ipv6
        family: ipv6_addr
        tiemout: 36h
[...]
...
```


### `map` properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#MAPS)

| Property name     | Description                                                                                                                             |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **`name`**        | Name of the map.                                                                                                                        |
| **`keys_type`**   | Type of the keys. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto` or `mark`.                        |
| **`values_type`** | Type of the values. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto`, `mark`, `counter` or `quota`.  |
| `elements`        | A list of elements contained in the map. Elements must conform to the map `keys_type` and `values_type`. See below.                     |

#### `map element` properties

| Property name   | Description                    |
| --------------- | ------------------------------ |
| **`key`**       | Key value.                     |
| **`value`**     | Value associated with the key. |

**Example:**

```yaml
---
nftables_tables:
[...]
    maps:
      - name: port2IP
        keys_type: inet_service
        values_type: ipv4_addr
        elements:
          - key: 22
            value: 192.168.0.45
          - key: 80
            value: 192.168.0.55
[...]
...
```


### `verdict_map` properties

A `verdict_map` is just a special case of `map` where the `values_type` is always `verdict`. As such, there is no `values_type` property. Also, elements contained in a `verdict_map` have a `verdict` property instead of the `value` property.

| Property name     | Description                                                                                                      |
| ----------------- | ---------------------------------------------------------------------------------------------------------------- |
| **`name`**        | Name of the map.                                                                                                 |
| **`keys_type`**   | Type of the keys. Must be either `ipv4_addr`, `ipv6_addr`, `ether_addr`, `inet_service`, `inet_proto` or `mark`. |
| `elements`        | A list of elements contained in the verdict map. See below                                                       |

#### `verdict_map element` properties

| Property name   | Description                      |
| --------------- | -------------------------------- |
| **`key`**       | Key value.                       |
| **`verdict`**   | Verdict associated with the key. |

Example:

```yaml
---
nftables_tables:
[...]
    verdict_maps:
      - name: allowed_ports
        keys_type: inet_service
        elements:
          - key: 22
            verdict: allow
          - key: 80
            verdict: allow
[...]
...
```

### `chain` properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#CHAINS)

| Property name | Description                                      |
| ------------- | ------------------------------------------------ |
| **`name`**    | Name of the chain.                               |
| **`base`**    | Base rule for the chain. See below.              |
| `rules`       | List of rules contained in the chain. See below. |

#### `base` properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#CHAINS)

| Property name  | Description                                                                    |
| -------------- | ------------------------------------------------------------------------------ |
| **`type`**     | The type of the chain. Must be either `filter`, `nat` or `route`.              |
| **`hook`**     | Hook where the chain is attached. Available values depend on `type`.           |
| **`priority`** | Integer determining the order of the chains attached to the same `hook`.       |
| `policy`       | Default policy for the chain. If specified, must be either `accept` or `drop`. |

#### `rule` properties

[DOC](https://manpages.debian.org/nftables/nftables.8.en.html#RULES)

| Property name   | Description                                              |
| --------------- | -------------------------------------------------------- |
| **`position`**  | Integer determining the order of the rules in the chain. |
| **`statement`** | Rule statement.                                          |
| `comment`       | A comment describing the rule.                           |

Example:

```yaml
---
nftables_tables:
[...]
    chains:
      - name: input
        base:
          type: filter
          hook: input
          priority: 0
          policy: drop
        rules:
          - position: 3
            statement: ip protocol icmp accept
            comment: Accept incoming ICMP.
          - position: 1
            statement: ct state established, related accept
          - position: 2
            statement: ct state invalid drop
[...]
...
```


## Contributing


