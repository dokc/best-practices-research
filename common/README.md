# Common Tests

This section covers the best practices to collect benchmarks baselines of the storage exposed in Kubernetes.

## FIO

FIO is a versatile tool for simulating various I/O workloads, allowing users to define custom scenarios
through its configurable parameters.

Page: https://fio.readthedocs.io/en/latest/fio_doc.html

A custom I/O scenario can be defined through a config file where parameters tune many aspects of the
I/O workload.

We recommend to prepare a few scenarios where writes and reads are separated, in order to gather
values on basic behaviours. Then you can create any other scenarios that suite your use cases the most.

Follow the official documentation for the complete set of parameters.

### FIO in Kubernetes

Once you have a working configuration file, you can create a configmap to be used by FIO deployment
in Kubernetes. For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: reads-test-fio
  namespace: benchmark
data:
  job: |-
    [read]
        direct=1
        bs=8k
        size=10G
        time_based=1
        runtime=300
        ioengine=libaio
        log_avg_msec=1000
        directory=/data
        readwrite=read
        write_bw_log=read
        write_lat_log=read
        write_iops_log=read
```

To actually run the FIO Pod is necessary to reserve a dedicated PVC specifying the Storage Class
to test, as shown below:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fio-benchmark
  namespace: benchmark
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi

# Select the storageClass to test
  storageClassName: <storage-class-name>
```

Finally the FIO Deployment should look like as follows:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fio-test
  namespace: benchmark
spec:
  selector:
    matchLabels:
      app.kubernetes.io/instance: fio-test
      app.kubernetes.io/name: fio
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: fio-test
        app.kubernetes.io/name: fio
    spec:

# nodeSelector is useful when testing local storage
# attached to the target node itself.
#
#      nodeSelector:
#        kubernetes.io/hostname: "<target-node-name>"

      containers:
        - name: fio
          image: wallnerryan/fiotools-aio:latest
          env:
            - name: JOBFILES
              value: /job/job.fio
            - name: PLOTNAME
              value: job
          ports:
            - containerPort: 8000
          readinessProbe:
            initialDelaySeconds: 60
            periodSeconds: 10
            tcpSocket:
              port: 8000
          volumeMounts:
            - mountPath: /data
              name: data
            - mountPath: /job
              name: job
            - mountPath: /tmp/fio-data
              name: tmp
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: fio-benchmark
        - configMap:
            items:
              - key: job
                path: job.fio
            name: reads-test-fio
          name: job
        - emptyDir: {}
          name: tmp
```

Note: the YAML mnanifest must be adjusted in accord to the test environment (i.e. taints, tolerations,
node selectors, and labels to select the correct node in case of local storage).

This deployment runs the All-In-One FIO image released by Ryan Wallner:
https://github.com/wallnerryan/fio-tools/

Once the container is in `Ready` state you can visit its web page by port forwarding the 8000 port:

```sh
kubectl port-forward -n benchmark deployments/fio-test 8000
```

The http://localhost:8000 page will show a combination of text and image (png) files: these are the
results of the benchmark test running that specific configmap. You will view them, and collect them
into a directory for later comparisons.

