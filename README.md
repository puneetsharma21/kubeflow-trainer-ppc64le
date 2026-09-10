# Kubeflow Trainer Examples for Power (ppc64le)

This repository contains **Kubeflow Trainer examples for Power (ppc64le)** architecture.

The examples cover PyTorch distributed training, multi-node training, checkpoint save/restore, worker suspend/resume, and basic Trainer runtime configuration.

The repository is intended for **testing, validation, and demonstration of Kubeflow Trainer workloads on ppc64le**.

---

## Repository Structure

```text
kubeflow-trainer-ppc64le/
│
├── index.html
│
├── runtimes/
│   ├── torch-distributed-cpu-ppc64le.yaml
│   ├── torch-distributed-cpu-ppc64le-2node.yaml
│   ├── torch-distributed-cpu-ppc64le-checkpoint.yaml
│
└── trainjobs/
    │
    ├── checkpoint/
    │   ├── torch-checkpoint-pvc.yaml
    │   └── torch-ddp-checkpoint-save-restore.yaml
    │
    ├── mnist/
    │   └── torch-ddp-mnist-2node.yaml
    │
    └── samples/
        ├── torch-cpu-ppc64le.yaml
        ├── torch-ddp-distributed-env-ppc64le.yaml
        ├── torch-save-resume-checkpoint.yaml
        └── torch-suspend-resume.yaml
```
mnist.ipynb link →  *[PyTorch DDP Fashion MNIST Training Example](https://puneetsharma21.github.io/kubeflow-trainer-ppc64le/)*
---

## Examples

### PyTorch Distributed MNIST

The `mnist/` directory contains a two-node PyTorch Distributed Data Parallel (DDP) example.

**Example:**

```text
trainjobs/mnist/torch-ddp-mnist-2node.yaml
```

This example demonstrates:

* Kubeflow Trainer `TrainJob`
* Multi-node PyTorch training
* PyTorch Distributed Data Parallel (DDP)
* Gloo backend
* Two worker nodes
* MNIST training
* Distributed process management using `torchrun`

The high-level workflow is:

```text
                  TrainJob
                     │
                     ▼
             Kubeflow Trainer
                     │
                     ▼
               Worker Pods
                     │
                     ▼
              torchrun / DDP
                     │
                     ▼
             Distributed Training
                     │
                     ▼
               MNIST Training
                     │
                     ▼
                Job Complete
```

---

## Checkpoint Save and Restore

The `checkpoint/` directory contains examples for persistent checkpoint storage and distributed training recovery.

### Checkpoint PVC

```text
trainjobs/checkpoint/torch-checkpoint-pvc.yaml
```

This creates the persistent storage used by the checkpoint workflow.

### Checkpoint Save and Restore

```text
trainjobs/checkpoint/torch-ddp-checkpoint-save-restore.yaml
```

This example demonstrates saving and restoring the training state during distributed PyTorch training.

The checkpoint can contain information such as:

* Model state
* Optimizer state
* Training epoch
* Training metadata

The checkpoint is stored on persistent storage rather than inside the worker pod.

The workflow is:

```text
                 TrainJob Starts
                       │
                       ▼
                 Worker Pods
                       │
                       ▼
             Distributed Training
                       │
                       ▼
                Save Checkpoint
                       │
                       ▼
              Persistent Storage
                       │
                       ▼
              Restore Checkpoint
                       │
                       ▼
             Restore Training State
                       │
                       ▼
             Continue Training
```

This allows the training application to restore its state after a worker is recreated, provided the application performs checkpoint restoration.

---

## Sample TrainJobs

The `samples/` directory contains smaller examples that demonstrate individual Kubeflow Trainer features and configurations.

### Basic PyTorch CPU Training

```text
trainjobs/samples/torch-cpu-ppc64le.yaml
```

A basic PyTorch training example targeting the ppc64le architecture.

### Distributed Environment

```text
trainjobs/samples/torch-ddp-distributed-env-ppc64le.yaml
```

Demonstrates the distributed environment provided to the training workers and how the training process uses the distributed configuration.

### Save and Resume Checkpoint

```text
trainjobs/samples/torch-save-resume-checkpoint.yaml
```

Demonstrates saving a training checkpoint and resuming training from the saved state.

### Suspend and Resume

```text
trainjobs/samples/torch-suspend-resume.yaml
```

Demonstrates suspending and resuming a Trainer workload.

---

## Training Runtimes

The `runtimes/` directory contains the Kubeflow Trainer runtime definitions used by the TrainJobs.

```text
runtimes/
├── torch-distributed-cpu-ppc64le.yaml
├── torch-distributed-cpu-ppc64le-2node.yaml
├── torch-distributed-cpu-ppc64le-checkpoint.yaml
```

The runtimes define the configuration required to launch the corresponding training workloads, including the training image and worker configuration.

The runtimes are designed for **Power (ppc64le)** compatible training images.

---

## Kubeflow Trainer Workflow

The examples follow the general Kubeflow Trainer workflow:

```text
                    TrainJob
                       │
                       ▼
              Kubeflow Trainer
                       │
                       ▼
           ClusterTrainingRuntime
                       │
                       ▼
                 Worker Pods
                       │
                       ▼
              Distributed Setup
                       │
                       ▼
                   torchrun
                       │
                       ▼
              PyTorch Training
                       │
                       ▼
          Checkpoint / Training State
```

Kubeflow Trainer manages the training workload and worker infrastructure, while the training application is responsible for model training and checkpoint handling.

---

## Running the Examples

### Prerequisites

Before running the examples, ensure that the cluster has:

* OpenShift
* Kubeflow Trainer
* A compatible PyTorch training runtime
* ppc64le worker nodes
* Persistent storage for checkpoint examples
* `oc` CLI configured with access to the cluster

Verify the Trainer installation:

```bash
oc get trainers -A
```

Verify the `TrainJob` CRD:

```bash
oc get crd trainjobs.trainer.kubeflow.org
```

---

### 1. Apply the Runtime

For example:

```bash
oc apply -f runtimes/torch-distributed-cpu-ppc64le-2node.yaml
```

Verify the runtime:

```bash
oc get clustertrainingruntimes
```

---

### 2. Create a TrainJob

For the MNIST example:

```bash
oc apply -f trainjobs/mnist/torch-ddp-mnist-2node.yaml
```

Check the TrainJob:

```bash
oc get trainjobs
```

Check the worker pods:

```bash
oc get pods
```

---

### 3. Check Training Logs

Find the worker pods:

```bash
oc get pods
```

Then view the logs:

```bash
oc logs <pod-name>
```

For multi-node training, logs from the individual worker pods can be used to verify distributed initialization and training progress.

---

## Checkpoint Example

Create the checkpoint PVC:

```bash
oc apply -f trainjobs/checkpoint/torch-checkpoint-pvc.yaml
```

Apply the checkpoint TrainJob:

```bash
oc apply -f trainjobs/checkpoint/torch-ddp-checkpoint-save-restore.yaml
```

Monitor the workload:

```bash
oc get trainjobs
oc get pods
```

Check the training logs:

```bash
oc logs <pod-name>
```

---

## Worker Failure and Recovery

The checkpoint examples can also be used to validate training recovery after a worker pod is recreated.

A simplified workflow is:

```text
                  Training
                     │
                     ▼
             Checkpoint Saved
                     │
                     ▼
             Worker Pod Deleted
                     │
                     ▼
              Worker Recreated
                     │
                     ▼
          Checkpoint Still Available
                     │
                     ▼
           Training State Restored
                     │
                     ▼
             Training Continues
```

The checkpoint is stored on persistent storage, so deleting a worker pod does not automatically remove the checkpoint.

The training application must explicitly load the checkpoint and restore the required training state.

---

## Suspend and Resume

The suspend/resume examples demonstrate controlling the lifecycle of a training workload.

Example runtime:

```text
runtimes/torch-suspend-resume.yaml
```

Example TrainJob:

```text
trainjobs/samples/torch-suspend-resume.yaml
```

The workflow can be represented as:

```text
             TrainJob Starts
                   │
                   ▼
                Running
                   │
                   ▼
               Suspended
                   │
                   ▼
                Resumed
                   │
                   ▼
                Running
                   │
                   ▼
              Job Complete
```

---

## Power (ppc64le)

These examples are specifically intended to validate Kubeflow Trainer workloads on **IBM(ppc64le)** architecture.

The repository provides coverage for:

| Area                    | Example                                  |
| ----------------------- | ---------------------------------------- |
| Basic PyTorch training  | `torch-cpu-ppc64le.yaml`                 |
| Distributed training    | `torch-ddp-distributed-env-ppc64le.yaml` |
| Multi-node MNIST        | `torch-ddp-mnist-2node.yaml`             |
| Checkpoint save/restore | `torch-ddp-checkpoint-save-restore.yaml` |
| Checkpoint storage      | `torch-checkpoint-pvc.yaml`              |
| Save/resume             | `torch-save-resume-checkpoint.yaml`      |
| Suspend/resume          | `torch-suspend-resume.yaml`              |

---

## Notebook

The repository also provides a rendered HTML notebook:

```text
index.html
```

The notebook can be viewed directly through GitHub Pages:

**[PyTorch DDP Fashion MNIST Training Example](https://puneetsharma21.github.io/kubeflow-trainer-ppc64le/)**
