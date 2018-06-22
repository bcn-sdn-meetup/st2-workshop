#

## ST2 Foundations

## Why

* <span style="color:yellow">Facilitated Troubleshooting</span> - triggering on system failures captured by  monitoring systems, running a series of diagnostic checks and posting results to a shared communication context.
* <span style="color:yellow">Automated remediation</span> - identifying and verifying failures, properly evacuating instances and notifying, but if anything goes wrong - freezing the workflow and paging a human.
* <span style="color:yellow">Continuous deployment</span> - build and test with CI tooling, provision a new infra, turn on some traffic with the load balancer, and roll-forward or roll-back, based on app performance data.


## How it works

![](https://docs.stackstorm.com/_images/architecture_diagram.jpg)

##  Concepts (I)

* <span style="color:yellow">Sensors</span> are Python plugins for either inbound or outbound integration that receives or watches for events respectively. When an event from external systems occurs and is processed by a sensor, a StackStorm trigger will be emitted into the system.
* <span style="color:yellow">Triggers</span> are StackStorm representations of external events. There are generic triggers (e.g. timers, webhooks) and integration triggers (e.g. Sensu alert, JIRA issue updated). A new trigger type can be defined by writing a sensor plugin.

##  Concepts (II)

* <span style="color:yellow">Actions</span> are StackStorm outbound integrations. There are generic actions (ssh, REST call), integrations (OpenStack, Docker, Puppet), or custom actions. Actions are either Python plugins, or any scripts, consumed into StackStorm by adding a few lines of metadata. Actions can be invoked directly by user via CLI or API, or used and called as part of rules and workflows.
* <span style="color:yellow">Rules</span> map triggers to actions (or to workflows), applying matching criteria and mapping trigger payload to action inputs.

##  Concepts (III)

* <span style="color:yellow">Workflows</span> stitch actions together into “uber-actions”, defining the order, transition conditions, and passing the data. Most automations are more than one-step and thus need more than one action. Workflows, just like “atomic” actions, are available in the Action library, and can be invoked manually or triggered by rules.
* <span style="color:yellow">Packs</span> are the units of content deployment. They simplify the management and sharing of StackStorm pluggable content by grouping integrations (triggers and actions) and automations (rules and workflows). A growing number of packs are available on StackStorm Exchange. Users can create their own packs, share them on Github, or submit to the StackStorm Exchange.

##  Concepts (IV)

* <span style="color:yellow">Audit</span> trail of action executions, manual or automated, is recorded and stored with full details of triggering context and execution results. It is also captured in audit logs for integrating with external logging and analytical tools: LogStash, Splunk, statsd, syslog.