## pod-restarter: restarts Pods in a failed state using client-go kubernetes packages

[![test](https://github.com/andreistefanciprian/pod-restarter-go/actions/workflows/test.yaml/badge.svg)](https://github.com/andreistefanciprian/pod-restarter-go/actions/workflows/test.yaml)

Performs the following steps:
* Looks for latest Pod Events that matches an Event Reason and Message
* If there are matching Pods, these Pods will go through a sequence of steps before they get deleted:
    - verify Pod exists
    - verify Pod has owner/controller
    - verify Pod has not been scheduled to be deleted
    - verify Pod is in a Failing State (Pending/Failed or Running with failing containers)
* If all above checks pass, Pod will be deleted

These steps are repeated in a loop on a polling interval basis.

### Configuring pod-restarter

pod-restarter is configurable through cli parameters.

#### `--polling-interval`
- Delete Pods that have matching Events with default Reason and Message every poll interval (seconds).
- Default value: 30 (seconds)

```
./pod-restarter --polling-interval 10
```

#### `--dry-run`
- Logs pod-restarter actions but don't actually delete any pods.
- Default value: disabled

```
./pod-restarter --dry-run
```

#### `--reason` and `--error-message`
- These parameters work together because every Event has a Reason and a related Message.
- These parameters are used for identifying failing Pods that match Event Reason and Message.
- Default values:
    - "FailedCreatePodSandBox" (Event Reason)
    - "container veth name provided (eth0) already exists" (Event Message)

```
# delete Pods that have Events with default Reason "FailedCreatePodSandBox" and default Message "container veth name provided (eth0) already exists"
./pod-restarter

# delete Pods that have Events with Reason "BackOff" and Message "Back-off pulling image"
./pod-restarter --reason "BackOff" --error-message "Back-off pulling image"
```

#### `--namespace`
- The kubernetes namespavce where pod-restarter should look for Failing Pods.
- Default value: "" (look for all namespaces)

```
# delete Pods in namespace default
./pod-restarter --namespace default
```

#### `--kubeconfig`
- When run locally (outside of cluster), specifies the kubeconfig config.
- Default value: ~/.kube/config

### Run and test on local machine/laptop

```
# install dependencies
go mod init
go mod tidy

# compile source code into executable binary
go build -o pod-restarter

# run in dry mode
./pod-restarter --dry-run --polling-interval 10

# run unit tests
go test -v ./...
```

### Deploy to k8s with helm

```
# build container image
task build

# deploy helm chart
task install

# verify helm release
helm list -A

# uninstall helm chart
task uninstall
```

### Simulate Failing Pods and test 

```
# generate failing pods due to pulling wrong images
cd infra/tests
bash generate_pending_pods.sh


# check app logs
kubectl logs -l app=pod-restarter -f
```