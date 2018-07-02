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

Install pack
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

```yaml
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

## Supported drivers

```
% /opt/stackstorm/virtualenvs/napalm/lib/python2.7/site-packages/napalm# cat _SUPPORTED_DRIVERS.py
SUPPORTED_DRIVERS = [
    "base",
    "eos",
    "ios",
    "iosxr",
    "junos",
    "nxos",
    "nxos_ssh",
]
```









