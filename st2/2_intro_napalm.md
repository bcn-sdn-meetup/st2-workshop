
#

## <span style="color:yellow">NAPALM</span>

Napalm [https://napalm-automation.net](https://napalm-automation.net/) is a <span style="color:yellow">vendor neutral</span>, cross-platform open source project that provides a unified <span style="color:yellow">API</span> to network devices.

![](https://avatars2.githubusercontent.com/u/16415577?s=400&v=4)

It works like a charm with Ansible, Salt and StackStorm (or as a Python library)

## Other projects

<span style="color:yellow">napalm-logs</span> is a Python library that listens to syslog messages from network devices and returns strucuted data following the OpenConfig or IETF YANG models.

<span style="color:yellow">napalm-yang</span> YANG (RFC6020) is a data modelling language, it’s a way of defining how data is supposed to look like. The napalm-yang library provides a framework to use models defined with YANG in the context of network management. It provides mechanisms to transform native data/config into YANG and vice versa.

## What it looks like

```
>>> from napalm import get_network_driver
>>> driver = get_network_driver('eos')
>>> with driver('localhost', 'vagrant', 'vagrant', optional_args={'port': 12443}) as device:
...     print device.get_facts()
...
{'os_version': u'4.15.2.1F-2759627.41521F', 'uptime': 2010, 'interface_list': [u'Ethernet1', u'Ethernet2', u'Management1'], 'vendor': u'Arista', 'serial_number': u'', 'model': u'vEOS', 'hostname': u'NEWHOSTNAME', 'fqdn': u'NEWHOSTNAME'}
```

## ... and under the hood?

```python
def get_facts(self):
    """Return facts of the device."""
    output = self.device.facts
    uptime = self.device.uptime or -1
    interfaces = junos_views.junos_iface_table(self.device)
    interfaces.get()
    interface_list = interfaces.keys()
    return {
        'vendor': u'Juniper',
        'model': py23_compat.text_type(output['model']),
        'serial_number': py23_compat.text_type(output['serialnumber']),
        'os_version': py23_compat.text_type(output['version']),
        'hostname': py23_compat.text_type(output['hostname']),
        'fqdn': py23_compat.text_type(output['fqdn']),
        'uptime': uptime,
        'interface_list': interface_list
    }
```
