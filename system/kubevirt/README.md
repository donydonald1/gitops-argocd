# KubeVirt Helm Chart

This chart installs KubeVirt and CDI (Containerized Data Importer) on Kubernetes.

## Overview

KubeVirt allows you to run Virtual Machines on Kubernetes alongside containerized workloads.

### Components Installed

1. **KubeVirt Operator** - Manages KubeVirt components
2. **CDI Operator** - Manages VM disk image imports
3. **KubeVirt CR** - Configures KubeVirt installation
4. **CDI CR** - Configures CDI installation

## Prerequisites

- Kubernetes 1.29+ (latest 3 releases supported)
- Hardware virtualization support (or enable software emulation)
- Privileged pods enabled
- Hugepages configured (recommended for production)

## Installation

The chart is deployed via ArgoCD as part of the system-apps ApplicationSet.

## Configuration

Key configuration options in `values.yaml`:

```yaml
kubevirt:
  version: "v1.4.0"
  configuration:
    developerConfiguration:
      useEmulation: false  # Set to true if no hardware virtualization
      featureGates:
        - LiveMigration
        - Snapshot
        - HotplugVolumes

cdi:
  enabled: true
  version: "v1.60.3"
```

## Usage

After installation, you can create VMs:

```yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: my-vm
spec:
  running: true
  template:
    spec:
      domain:
        devices:
          disks:
            - name: rootdisk
              disk:
                bus: virtio
          interfaces:
            - name: default
              masquerade: {}
        resources:
          requests:
            memory: 1Gi
      networks:
        - name: default
          pod: {}
      volumes:
        - name: rootdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
```

## Monitoring

ServiceMonitors are created for Prometheus integration when `monitoring.enabled: true`.

## Notes

- CRDs are managed by the operators themselves
- The chart does NOT require OLM (Operator Lifecycle Manager)
- For production, ensure hardware virtualization is available
