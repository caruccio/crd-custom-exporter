# CRD Custom Exporter

Expose custom CRD fields as prometheus metrics

# Proposal

Disclaimer: this is my very first kubernetes focused project, so be gentle please :)

This is an initial proposal to expose Kubernetes' [Custom Resource Definitions (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) as [Prometheus](https://prometheus.io) metrics endpoints. The aim is to monitore and alert on whatever an Custom Resource (CR) exposes, by using a simple `property -> metric` mapping model.

Supose we have the following [Velero](https://github.com/heptio/velero/) Schedule CR (incomplete for readability):

```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: shopping
  namespace: megastore
spec:
  schedule: 0 3 * * *
  template:
    includeClusterResources: true
    includedNamespaces:
    - webapp
    - database
    includedResources:
    - deployments
    - cronjobs
```

In order to expose metrics regarding `includedNamespaces`, one could define the following config:

```yaml
customResourceDefinitions:
- apiVersion: velero.io/v1
  kind: Schedule
  inspect:
  - included_namespaces: spec.template.includedNamespaces
```

or maybe:

```yaml
customResourceDefinitions:
- apiVersion: velero.io/v1
  kind: Schedule
  spec:
    template:
      includedNamespaces:
      - webapp
      - database
      - non-existing
```

Which will generate the following prometheus metrics:

```
crd_exporter_velero_io_v1_included_namespaces_webapp{namespace="megastore", name="shopping"} 1
crd_exporter_velero_io_v1_included_namespaces_database{namespace="megastore" ,name="shopping"} 1
```

or maybe

```
crd_exporter_schedule_velero_io_included_namespaces_webapp{namespace="megastore", name="shopping", crd="schedule", api_group="velero.io", api_version="v1"} 1
crd_exporter_schedule_velero_included_namespaces_database{namespace="megastore", name="shopping", crd="schedule", api_group="velero.io", api_version="v1"} 1
```

or maybe

```
crd_exporter_schedule_velero_io{namespace="megastore", name="shopping", crd="schedule", api_group="velero.io", api_version="v1", path="spec.template.includedNamespace[0].webapp"} 1
crd_exporter_schedule_velero_io{namespace="megastore", name="shopping", crd="schedule", api_group="velero.io", api_version="v1", path="spec.template.includedNamespace[1].database"} 1
crd_exporter_schedule_velero_io{namespace="megastore", name="shopping", crd="schedule", api_group="velero.io", api_version="v1", path="spec.template.includedNamespace[2].non-existing"} 0
```
