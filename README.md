# CRD Custom Exporter

Expose custom CRD fields as prometheus metrics

# Proposal

Disclaimer: this is my very first kubernetes focused project, so be gentle please :)

This is an initial proposal to expose Kubernetes' [Custom Resource Definitions (CRD)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) as [Prometheus]() [metrics endpoints](). The aim is to monitore and alert on whatever a Custom Resource (CR) exposes using a simple mapping `property -> metric` model.
