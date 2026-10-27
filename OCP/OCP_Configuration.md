### (26A) OpenShift (OCP) Configuration Guide
post-installation configuration procedures for OpenShift (OCP)

#### 1. Move Infrastructure Pods to Master Nodes (2-1)
To optimize resource utilization and dedicate worker nodes exclusively to applications, infrastructure pods can be migrated to run on the master nodes.

```
+-------------------------------------------------------------+
|                 OpenShift Cluster Nodes                     |
|                                                             |
|   +-----------------------+     +-----------------------+   |
|   |      Master Node      |     |      Worker Node      |   |
|   |                       |     |                       |   |
|   |  [Inbound Traffic]    |     |  [User Application]   |   |
|   |           v           |     |  [Heavy Workloads]    |   |
|   |  [Ingress Router Pod] |     |                       |   |
|   +-----------------------+     +-----------------------+   |
+-------------------------------------------------------------+
```
```
# Patch ingress controller node placement rules to schedule on master nodes
oc patch ingresscontroller default -n openshift-ingress-operator --type=merge -p '{"spec":{"nodePlacement":{"nodeSelector":{"matchLabels":{"node-role.kubernetes.io/master":""}}}}}'
```
#### 2. Set Master Node Schedulable to False (2-2)
To prevent general user workloads from scheduling onto master nodes and ensure overall cluster stability, mark master nodes as unschedulable (cordon).

```
# Mark master nodes as unschedulable
oc adm cordon <master-node-name>
```
#### 3. NTP Server Configuration (2-3)
. Configure Chrony to synchronize cluster nodes with a trusted internal NTP server.

```Bash
# Edit chrony configuration on cluster nodes
cat <<EOF > /etc/chrony.conf
server <ntp-server-ip> iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
EOF

systemctl restart chronyd
systemctl enable chronyd
```
#### 4. Creating an OCP Administrative Account (2-4)
Establish a cluster-admin or custom administrative account using an identity provider like HTPasswd for secure cluster management.

```Bash
# Create a htpasswd secret for identity provider authentication
oc create secret generic htpasswd-secret --from-file=htpasswd=/etc/openshift/htpasswd -n openshift-config

# Apply OAuth configuration referencing the htpasswd provider
oc apply -f - <<EOF
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: htpasswd_idp
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd-secret
EOF
```

#### 5. Helm CLI Setup (2-5)
Install and configure the Helm package manager to deploy charts and packages within the OCP cluster.
```
# Download and install Helm CLI
curl -fsSL https://get.helm.sh/helm-v3.14.0-linux-amd64.tar.gz -o helm.tar.gz
tar -zxvf helm.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
chmod +x /usr/local/bin/helm
```
#### 6. Container Registry Configuration (2-6)
Configure cluster-wide image registries, trust certificates, and mirror sources for offline/disconnected enterprise environments
```
# Configure ImageDigestMirrorSet for target private registry (Harbor)
oc apply -f - <<EOF
apiVersion: config.openshift.io/v1
kind: ImageDigestMirrorSet
metadata:
  name: harbor-mirror-set
spec:
  imageMirrors:
  - source: registry.redhat.io
    mirrors:
    - <harbor-url>/ocp/redhat
EOF
```

#### 7. Change Kubernetes Namespace Mount Options (2-7)
Customize default storage and mount permissions for specific namespaces to align with security context constraints (SCC) or storage driver rules.
```
# Annotate namespace to bypass or modify security context restrictions
oc annotate namespace default openshift.io/sa.scc.pod-security-review-ignore=true
```

#### 8. Change Default Container Runtime: runc to crun (2-8)
Transition the container runtime from `runc` to `crun` for enhanced performance, low memory overhead, and speed improvements, especially beneficial for heavy workloads.
```
# Apply ContainerRuntimeConfig custom resource to switch runtime to crun for worker nodes
oc apply -f - <<EOF
apiVersion: machineconfiguration.openshift.io/v1
kind: ContainerRuntimeConfig
metadata:
  name: set-crun-runtime
spec:
  machineConfigPoolSelector:
    matchLabels:
      pools.operator.machineconfiguration.openshift.io/worker: ""
  containerRuntimeConfig:
    defaultRuntime: crun
EOF
```
