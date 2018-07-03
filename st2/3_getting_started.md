
#

## Getting started

Register on Arista Website (https://www.arista.com/en/user-registration)

Download Arista vEOS (https://www.arista.com/en/support/software-download) and add it as a Vagrant box
```
vagrant box add --name veos ./vEOS-lab-4.20.1F-virtualbox.box
```

## Basic ST2 usage

ST2 Authentication
```
% export ST2_AUTH_TOKEN=`st2 auth -t -p 'Ch@ngeMe' st2admin`
% st2 login st2admin --password 'Ch@ngeMe' --write-password
```

Run a local shell command
```
% st2 run core.local -- uname -r
```

## Let's use Napalm pack

Install pack (already there):
```
% st2 pack install napalm
```

List Action list
```
% st2 action list --pack=napalm
+--------------------------------------+--------+---------------------------------------------------------------+
| ref                                  | pack   | description                                                   |
+--------------------------------------+--------+---------------------------------------------------------------+
| napalm.bgp_prefix_exceeded_chain     | napalm | Action Chain to process a BGP neighbor prefix limit exceeded  |
|                                      |        | event.                                                        |
| napalm.check_consistency             | napalm | Check that the device's configuration is consistent with the  |
|                                      |        | 'golden' config in a Git repository                           |
| napalm.cli                           | napalm | Run CLI commands on a device using NAPALM.                    |
```

## Configure NAPALM authentication

It's already provided for you in /opt/stackstorm/configs/napalm.yaml

```
---
html_table_class: napalm

credentials:
  core:
    username: stanley
    password: stanley 

devices:
- hostname: 192.168.100.11
  driver: eos
  credentials: core
- hostname: 192.168.100.12 
  driver: eos 
  credentials: core 
```

## Apply configuration changes

Every time you change configuration you should reapply it:

```
% st2ctl reload --register-configs

Registering content...[flags = --config-file /etc/st2/st2.conf --register-configs]
2018-06-27 20:21:07,291 INFO [-] Connecting to database "st2" @ "127.0.0.1:27017" as user "stackstorm".
2018-06-27 20:21:07,689 INFO [-] =========================================================
2018-06-27 20:21:07,690 INFO [-] ############## Registering configs ######################
2018-06-27 20:21:07,691 INFO [-] =========================================================
2018-06-27 20:21:07,896 INFO [-] Registered 1 configs.
##### st2 components status #####
st2actionrunner PID: 1101
st2actionrunner PID: 1147
```

## First command, get data

```
st2 run napalm.get_facts hostname=192.168.100.11
.
id: 5b3a02c902ebd506863c8e81
status: succeeded
parameters:
  hostname: 192.168.100.11
result:
  exit_code: 0
  result:
    raw:
      fqdn: localhost
      hostname: localhost
      interface_list:
      - Ethernet1
      - Management1
      model: vEOS
      os_version: 4.20.1F-6820520.4201F
      serial_number: ''
      uptime: 10991
      vendor: Arista
  stderr: ''
  stdout: ''
```

## Submit configuration

```
st2 run napalm.loadconfig hostname=192.168.100.11 config_text="hostname something"
..
id: 5b3a02c102ebd506863c8e7e
status: succeeded
parameters:
  config_text: hostname something
  hostname: 192.168.100.11
result:
  exit_code: 0
  result: load (merge) successful on 192.168.100.11
  stderr: ''
  stdout: ''
```

## Actions

Build custom actions: [https://docs.stackstorm.com/actions.html](https://docs.stackstorm.com/actions.html)

To register a new action:

* Place it into the content location.
* Tell the system that the action is available.
* The actions are grouped in packs and located at /opt/stackstorm/packs

## 

```
cat /opt/stackstorm/packs/napalm/actions/cli.py
```

```python
from lib.action import NapalmBaseAction
class NapalmCLI(NapalmBaseAction):
    """Run CLI commands on a network device via NAPALM
    """
    def run(self, commands, **std_kwargs):

        with self.get_driver(**std_kwargs) as device:
            cmds_output = device.cli(commands)
            result = {'raw': cmds_output}
            result_with_pre = {}
            result_as_array = {}
            for this_cmd in cmds_output:
                result_as_array[this_cmd] = cmds_output[this_cmd].split('\n')
                if self.htmlout:
                    result_with_pre[this_cmd] = "<pre>" + cmds_output[this_cmd] + "</pre>"
            result['raw_array'] = result_as_array
            if self.htmlout:
                result['html'] = self.html_out(result_with_pre)
        return (True, result)
```


## Workflow

* Actions, by design, are intended to perfom a single task well
* However, in real world, we usually run several discrte tasks, and includes some decision making along the way -> Workflows
* **Mistral** is an OpenStack project that provides (included in ST2):
    * A standarised YAML-based language for defining workflows
    * Open source software for receiving and processing workflows execution requests 


## Example Workflow

```
--- 
version:'2.0'
napalm.interface_down_workflow:
  input: 
    - hostname
    - interface
  type: direct
  tasks:
    show_interface:
      action: "napalm.get_interfaces"
      input:
        hostname: "{{ _.hostname }}"
        interface: "{{ _interface}}"
      on-success: "show_interface_counters"
    show_interface_counters:
      action: "napalm.get_interfaces"
      input:
        hostname: "{{ _.hostname }}"
        interface: "{{ _interface}}"
        counters: true
      on-success: "show_log"
    show_log:
      action: "napalm.get_log"
      input:
        hostname: "{{ _.hostname }}"
        lastlines: 10
```

##

```
st2 run napalm.interface_down_workflow hostname=192.168.100.11 interface=Ethernet1 message=debug
..
id: 5b3a913902ebd506863c8ea4
action.ref: napalm.interface_down_workflow
parameters:
  hostname: 192.168.100.11
  interface: Ethernet1
  message: debug
status: failed
result_task: show_log
result:
  exit_code: 1
traceback: None
failed_on: show_log

+--------------------------+------------------------+--------------+--------------+-----------------+
| id                       | status                 | task         | action       | start_timestamp |
+--------------------------+------------------------+--------------+--------------+-----------------+
| 5b3a913902ebd506863c8ea7 | succeeded (1s elapsed) | show_interfa | napalm.get_i | Mon, 02 Jul     |
|                          |                        | ce           | nterfaces    | 2018 20:55:21   |
|                          |                        |              |              | UTC             |
| 5b3a913a02ebd506863c8ea9 | succeeded (1s elapsed) | show_interfa | napalm.get_i | Mon, 02 Jul     |
|                          |                        | ce_counters  | nterfaces    | 2018 20:55:22   |
|                          |                        |              |              | UTC             |
| 5b3a913c02ebd506863c8eab | failed (1s elapsed)    | show_log     | napalm.get_l | Mon, 02 Jul     |
|                          |                        |              | og           | 2018 20:55:23   |
|                          |                        |              |              | UTC             |
+--------------------------+------------------------+--------------+--------------+-----------------+

```

##

```
st2 execution get 5b3a913a02ebd506863c8ea9
id: 5b3a913a02ebd506863c8ea9
status: succeeded (1s elapsed)
parameters:
  counters: true
  hostname: 192.168.100.11
  htmlout: true
  interface: Ethernet1
result:
  exit_code: 0
  result:
    html: <table class="napalm"><tr><th>tx_multicast_packets</th><td>495</td></tr><tr><th>tx_discards</th><td>0</td></tr><tr><th>tx_octets</th><td>234450</td></tr><tr><th>tx_errors</th><td>0</td></tr><tr><th>rx_octets</th><td>167884</td></tr><tr><th>tx_unicast_packets</th><td>490</td></tr><tr><th>rx_errors</th><td>0</td></tr><tr><th>tx_broadcast_packets</th><td>0</td></tr><tr><th>rx_multicast_packets</th><td>184</td></tr><tr><th>rx_broadcast_packets</th><td>50</td></tr><tr><th>rx_discards</th><td>0</td></tr><tr><th>rx_unicast_packets</th><td>487</td></tr><tr><th>name</th><td>Ethernet1</td></tr></table>
    raw:
      name: Ethernet1
      rx_broadcast_packets: 50
      rx_discards: 0
      rx_errors: 0
      rx_multicast_packets: 184
      rx_octets: 167884
      rx_unicast_packets: 487
      tx_broadcast_packets: 0
      tx_discards: 0
      tx_errors: 0
      tx_multicast_packets: 495
      tx_octets: 234450
      tx_unicast_packets: 490
  stderr: ''
  stdout: ''

```

##

```
st2 execution get 5b3a913c02ebd506863c8eab
id: 5b3a913c02ebd506863c8eab
status: failed (1s elapsed)
parameters:
  hostname: 192.168.100.11
  htmlout: true
  lastlines: 10
result:
  exit_code: 1
  result: None
  stderr: "Traceback (most recent call last):
  File "/opt/stackstorm/runners/python_runner/python_runner/python_action_wrapper.py", line 320, in <module>
    obj.run()
  File "/opt/stackstorm/runners/python_runner/python_runner/python_action_wrapper.py", line 179, in run
    output = action.run(**self._parameters)
  File "/opt/stackstorm/packs/napalm/actions/get_log.py", line 40, in run
    cmd_result = device.cli(commands)
  File "/opt/stackstorm/virtualenvs/napalm/lib/python2.7/site-packages/napalm/eos/eos.py", line 644, in cli
    raise CommandErrorException(str(cli_output))
napalm.base.exceptions.CommandErrorException: {u'term width 32767': u'Invalid command: "term width 32767"'}
"
  stdout: ''
```

## Sensors and Triggers

* Sensors -> Pull
* Triggers -> Push (webhooks)

```
st2 sensor list --pack napalm
+-----------------------+--------+-----------------------+---------+
| ref                   | pack   | description           | enabled |
+-----------------------+--------+-----------------------+---------+
| napalm.NapalmLLDPSens | napalm | Sensor that uses      | False   |
| or                    |        | NAPALM to retrieve    |         |
|                       |        | LLDP information from |         |
|                       |        | network devices       |         |
+-----------------------+--------+-----------------------+---------+

st2 trigger list --pack napalm
+-----------------------------+--------+------------------------------+
| ref                         | pack   | description                  |
+-----------------------------+--------+------------------------------+
| napalm.LLDPNeighborDecrease | napalm | Trigger which occurs when a  |
|                             |        | device's LLDP neighbors      |
|                             |        | decrease                     |
+-----------------------------+--------+------------------------------+

```

##

```
% st2 sensor enable napalm.NapalmLLDPSensor

+---------------+--------------------------------------------------------------+
| Property      | Value                                                        |
+---------------+--------------------------------------------------------------+
| id            | 5b39c90d02ebd507dacc6a42                                     |
| name          | NapalmLLDPSensor                                             |
| pack          | napalm                                                       |
| description   | Sensor that uses NAPALM to retrieve LLDP information from    |
|               | network devices                                              |
| artifact_uri  | file:///opt/stackstorm/packs/napalm/sensors/lldp_sensor.py   |
| enabled       | True                                                         |
| entry_point   | sensors.lldp_sensor.NapalmLLDPSensor                         |
| ref           | napalm.NapalmLLDPSensor                                      |
| trigger_types | [                                                            |
|               |     "napalm.LLDPNeighborDecrease",                           |
|               |     "napalm.LLDPNeighborDecrease"                            |
|               | ]                                                            |
| uid           | sensor_type:napalm:NapalmLLDPSensor                          |
+---------------+--------------------------------------------------------------+

tail -f /var/log/st2/st2sensorcontainer.log
```

## Rules

Rules is the key part of ST2, it brings together actions and sensors and triggers to create event driven automation.

[https://docs.stackstorm.com/rules.html](https://docs.stackstorm.com/rules.html)

```
% st2 rule list

+-----------------------+---------+-----------------------+---------+
| ref                   | pack    | description           | enabled |
+-----------------------+---------+-----------------------+---------+
| chatops.notify        | chatops | Notification rule to  | True    |
|                       |         | send results of       |         |
|                       |         | action executions to  |         |
|                       |         | stream for chatops    |         |
| napalm.bgp_prefix_exc | napalm  | Webhook which handles | True    |
| eeded                 |         | a BGP neighbor        |         |
|                       |         | exceeding it's prefix |         |
|                       |         | limit                 |         |
| napalm.configuration_ | napalm  | Webhook which handles | True    |
| change                |         | when a device has     |         |
|                       |         | been configured and   |         |
|                       |         | changes commited      |         |
| napalm.interface_down | napalm  | Webhook which handles | True    |
|                       |         | an interface going    |         |
|                       |         | down on a network     |         |
|                       |         | device.               |         |
| napalm.lldp_remediate | napalm  | Demonstrate simple    | True    |
|                       |         | auto-remediation      |         |
|                       |         | event                 |         |
+-----------------------+---------+-----------------------+---------+

```

##

```
---
name: "lldp_notify"
pack: "napalm"
enabled: true
description: "Notify of LLDP Neighbor Decrease"

trigger:
  type: "napalm.LLDPNeighborDecrease"
  parametres: {}

criteria: {}

action:
  ref: slack.post_message
  parameters:
    message: "WARNING: {{trigger.device}}'s LLDP Neighbors just went DOWN to {{trigger.newpeers}} (was {{trigger.oldpeers}})"
    channel: '#general'
```

## Managing rules

* To deploy a rule, use the CLI command: `st2 rule create ${PATH_TO_RULE}`,
    * `st2 rule create /usr/share/doc/st2/examples/rules/sample_rule_with_webhook.yaml`
* To reload all the rules, use `st2ctl reload --register-rules`.
* Custom rules can be placed in any accessible folder on local system. By convention, custom rules are placed in the /opt/stackstorm/packs/<pack_name>/rules directory.
* To make testing rules easier we provide a `st2-rule-tester` tool which can evaluate rules against trigger instances without running any of the StackStorm components.
