# Datadog

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Examples](#examples)

This roles installs the Datadog agent.

The agent service is intentionally disabled on bootup; during boot a machine will most likely
consume a high amount of resources whilst the OS and all of it's services start which will trip any
CPU or Memory usage alarms erroneously.




## ansible.cfg

This role is designed to work with merge "hash_behaviour". Make sure your
ansible.cfg contains these settings

```INI
[defaults]
hash_behaviour = merge
```




## Installation and Dependencies

To install run `ansible-galaxy install cloudposse.datadog` or add this to your
`roles.yml`.

```YAML
- name: cloudposse.datadog
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`




## Tags

* `build` - Installs Datadog and all it's dependencies.
* `configure` - Configures datadog and install integrations




## Integrations

The `datadog/templates/conf.d` directory contains the implemented integrations. The majority of them are self explanatory.

#### TCP Check

Sample data structure for TCP Check integration.

```
roles:
  - role: sansible.datadog
    datadog:
      integrations:
        tcp_check:
          - endpoint: "my.domain.com"
            name: "TCP Check for my.domain.com"
            port: "443"
          - endpoint: "your.domain.com"
            name: "TCP Check for your.domain.com"
            port: "80"
            timeout: 2
            collect_response_time: "false"
            min_collection_interval: 60
      tags:
        - some_app
        - role:some_app
```




## Examples

To install:

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  roles:
    - role: sansible.datadog
```

Setup Datadog with some integrations and tags:

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  roles:
    - role: sansible.datadog
      datadog:
        integrations:
          php_fpm: { }
          nginx: { }
        tags:
          - some_app
          - role:some_app
```

You can disable the agent completely if desired, this can be used to build images
that have DD installed with the option turn DD off in certain environments where
monitoring is not needed (eg. dev or scratch environments):

```YAML
# Environment var file eg. vars/dev/vars.yml

datadog:
  enabled: no
```

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  vars_files:
    - vars/dev/vars.yml

  roles:
    - role: sansible.datadog
```

**Note** This behaviour requires hash_behaviour to be set to merge.

By default this role will start the DD agent in the [tasks/configure.yml] task file,
this behaviour can be disabled however if you wish to start the agent within
another role:

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  vars_files:
    - vars/dev/vars.yml

  roles:
    - role: sansible.datadog
      datadog:
        autostart_agent: no

    # The task of this role will be be to start the DD agent service
    role: my_service_role
```

Using .default_tags to set global tags, these get blended with .tags:

```YAML
# Environment var file eg. vars/dev/vars.yml

datadog:
  default_tags:
    - environment:dev
```

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  vars_files:
    - vars/dev/vars.yml

  roles:
    - role: sansible.datadog
      datadog:
        tags:
          - some_app
          - role:some_app
```

**Note** This behaviour requires hash_behaviour to be set to merge.

This will result in the following tags:

```
- environment:dev
- some_app
- role:some_app
```

Disable Datadog service (useful for disabling in some environments):

```YAML
- name: Install and configure Datadog
  hosts: "somehost"

  roles:
    - role: sansible.datadog
      datadog:
        enabled: false
```
