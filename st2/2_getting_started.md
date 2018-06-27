#

## Getting started

ST2 Authentication
```
% export ST2_AUTH_TOKEN=`st2 auth -t -p 'Ch@ngeMe' st2admin`
% st2 login st2admin --password 'Ch@ngeMe' --write-password
```

Run a local shell command
```
% st2 run core.local -- uname -r
```

Run a shell command on remote hosts. Requires passwordless SSH configured.
```
% st2 run core.remote hosts='192.168.100.11' username='stanley' -- uname -r
```

## Configure st2 authentication

There is a user (stanley) already provisioned in ST2 with a preconfigured key in /home/stanley/.ssh/stanley_rsa.pub
In both VyOS routers we should create the user with the proper public key

```
set system login user stanley authentication public-keys key-stanley-pub key "public key base64"
set system login user stanley authentication public-keys key-stanley-pub type ssh-rsa
commit
```

In ST2, now it works!

```bash
% st2 run core.remote hosts='192.168.100.11' username='stanley' -- uname
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

In /opt/stackstorm/configs/napalm.yaml

```yaml
---
html_table_class: napalm

credentials:
  local:
    username: vagrant
    password: vagrant

devices:
  - hostname: 192.168.100.11
    driver: vyos
    credentials: local
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









