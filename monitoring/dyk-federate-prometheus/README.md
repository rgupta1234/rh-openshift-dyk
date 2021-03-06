# DO YOU KNOW - How to federate critical metrics from the Prometheus shipped with OpenShift Monitoring

In OpenShift 3.11, Red Hat deploys an out-of-the-box cluster-level Prometheus monitoring ecosystem including Prometheus, Alertmanager, Grafana, 
and various exporters as well as pre-packaged alerting rules and Grafana dashboards. To ensure successful over-the-air updates, this ecosystem
has only limited configuration possibilities. For example, you cannot add new alerting rules or Grafana dashboards.

In this repository, I am going to highlight one option to work around that limitation and also enable other use cases such as correlating app
and cluster metrics.

Federation enables people to scrape specific metrics from the Cluster Monitoring Prometheus by configuring a "federated Prometheus". That 
Prometheus can be used for all the custom use cases.

**Please be aware that this setup only works for 3.11 and some bits and pieces will most probably change with OpenShift 4.0. Hence, the OpenShift team does not offer any support for a federated setup targeting the OOTB Cluster Monitoring Prometheus.**

The core pieces to make this work with the OAuth proxy deployed in any OpenShift cluster are:
* ClusterRoleBinding that enables access to the `openshift-monitoring` namespace where the Cluster Monitoring Prometheus is deployed. I am reusing an existing ClusterRole created by the monitoring stack.
* A Role deployed to the `openshift-monitoring` that enables access to resources for the "federated Prometheus" to discover the Cluster Monitoring Prometheus service.
* A ServiceMonitor that configures the `/federate` target including the TLS config to get past the OAuth proxy.

I am using the [Prometheus Operator](https://github.com/coreos/prometheus-operator) for the ease of configuring all the different objects. But you don't need to use it and instead create a deployment and a target configuration.

## Installation

```
oc create namespace monitoring
oc apply -f manifests
```

## Known limitation:

The Cluster Monitoring stack deployes two instances of Prometheus and both are going through a single service. Hence, the "federate Prometheus" will constantly switch between both instances creating small scraping gaps. This is still a problem and currently under investigation. Again, this is another reason that this setup is not supported by the Red Hat team. A workaround is to use a target configuration instead of a `ServiceMonitor` CRD and configure scraping from a specific Prometheus instance instead of the service.