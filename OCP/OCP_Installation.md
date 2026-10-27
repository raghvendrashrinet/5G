###  OpenShift (OCP) Installation, Configuration, and Image Mirroring Architecture Guide

#### 1. Executive Overview & Workflow Architecture
In disconnected or secure enterprise environments (such as Core Cloud deployments), direct internet access to the Red Hat Registry is restricted from production target nodes. To bridge this gap, an intermediate Common Server with internet access downloads the required OCP release images and operators, packages them into an offline archive file, transfers them securely across boundaries, and publishes them into a local target Harbor registry.

```
+----------------------+      1. Download Images      +-----------------------+
|  Red Hat Registry    | ---------------------------> |   Common Server       |
|  (registry.redhat.io)|                              |   (Internet-Enabled)  |
+----------------------+                              +-----------------------+
                                                                 |
                                                                 | 2. oc-mirror
                                                                 v  (--config)
                                                      +-----------------------+
                                                      | Archive Bundle        |
                                                      | (mirror_seq1_...tar)  |
                                                      +-----------------------+
                                                                 |
                                                                 | 3. Transfer / Hand-carry
                                                                 v  (Isolated Network)
+----------------------+      5. Verify Image Pull    +-----------------------+
|   Target Harbor      | <--------------------------- |   Target Cluster      |
|   (Local Registry)   |    4. oc-mirror (--from)     |   Environment         |
+----------------------+                              +-----------------------+

```

#### . Prerequisites and Environment Setup
- Host Operating System: Recommended Red Hat Enterprise Linux 9.x (RHEL 9) on the Common Server.
- Required Tooling: openshift-client and oc-mirror matching the target OpenShift version (e.g., version 4.20).
- Authentication: Valid Red Hat pull-secret configured in the Docker configuration path.

Component,Source / Reference Path,Target Location
openshift-client,openshift-mirror Client,/usr/local/bin
oc-mirror CLI,openshift-mirror Client,/usr/local/bin
Pull Secret,Red Hat Pull-Secret,/root/.docker/config.json

#### 3. Step-by-Step Implementation Guide

##### Step 3.1: Install OpenShift Client & oc-mirror Binaries
Download and extract the necessary command-line utilities onto your Common Server
```
## Download openshift-client and oc-mirror binaries for version 4.20.8
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.20.8/openshift-client-linux-amd64-rhel9-4.20.8.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.20.8/oc-mirror.rhel9.tar.gz

## Extract binaries to system path
tar -xvf openshift-client-linux-amd64-rhel9-4.20.8.tar.gz -C /usr/local/bin
tar -xvf oc-mirror.rhel9.tar.gz -C /usr/local/bin
chmod 755 /usr/local/bin/oc-mirror
```
##### Step 3.2: Configure Pull Secret Authentication
Establish authentication credentials so the local tooling can interface securely with Red Hat registries:
```
## Copy the pull-secret and create pull-secret file
vi /root/pull-secret.json

## Set the created pull-secret file as the Docker Config of the Server
ln -s /root/pull-secret.json /root/.docker/config.json
```
##### Step 3.3: Define ImageSetConfiguration Manifest
Create an imagesetconfig.yaml file to specify exact OCP channels, platform architectures, and core operators:
```
mkdir ocp_mirror && cd ocp_mirror
vi imagesetconfig.yaml
```

Configuration Content:
```
kind: ImageSetConfiguration
apiVersion: mirror.openshift.io/v1alpha2
mirror:
  platform:
    architectures:
      - "amd64"
    channels:
      - name: stable-4.20
        type: ocp
        minVersion: 4.20.14
        maxVersion: 4.20.14
        shortestPath: true
        graph: false
  operators:
    - catalog: registry.redhat.io/redhat/redhat-operator-index:v4.20
      packages:
        - name: servicemeshoperator
          channels:
            - name: stable
              minVersion: 2.6.11-0
              maxVersion: 2.6.11-0
        - name: kiali-ossm
          channels:
            - name: stable
              minVersion: 2.17.2
              maxVersion: 2.17.2
        - name: tempo-product
          channels:
            - name: stable
              minVersion: 0.19.0-2
              maxVersion: 0.19.0-2
        - name: cluster-logging
          channels:
            - name: stable-6.4
              minVersion: 6.4.0
              maxVersion: 6.4.0
        - name: loki-operator
          channels:
            - name: stable-6.4
              minVersion: 6.4.0
              maxVersion: 6.4.0
        - name: sriov-network-operator
          channels:
            - name: stable
              minVersion: 4.20.0-202512081147
              maxVersion: 4.20.0-202512081147
        - name: openshift-gitops-operator
          channels:
            - name: gitops-1.19
              minVersion: 1.19.0
              maxVersion: 1.19.0
    - catalog: registry.redhat.io/redhat/community-operator-index:v4.20
      packages:
        - name: grafana-operator
          channels:
            - name: v5
              minVersion: 5.21.2
              maxVersion: 5.21.2
```
##### Step 3.4: Execute Image Download & Packaging
Run the oc-mirror utility in stateless mode to pull release images and generate the compressed sequence archive:
```
oc-mirror --config=./imagesetconfig.yaml file://mirror/ --source-skip-tls --skip-missing --skip-verification --skip-image-pin
```
Upon completion, an archive bundle named `mirror_seq1_000000.tar` is generated inside the mirror directory.

##### 4. Harbor Upload & Verification Pipeline
```
[ Harbor Archive (.tar) ] ---> [ Docker Login to Harbor ] ---> [ oc-mirror --from ] ---> [ Registry Push & Verify ]
```
Step 4.1: Configure Local Host Resolution
Update /etc/hosts to map your target Harbor IP and URL:
```
vi /etc/hosts
# Add Harbor IP and URL:
[Harbor_IP] [Harbor_URL]
```

Step 4.2: Authenticate and Push Images to Target Harbor
Log in to your private Harbor instance and push the archived image set using the --from flag:

```
## Log in to Harbor
docker login -u [[harbor-id]] -p [[harbor-password]] [[harbor-url]]

## Push images from archive to Harbor
oc-mirror --from=./mirror_seq1_000000.tar docker://[[harbor-url]]/ocp --dest-skip-tls
```

Step 4.3: Final Validation and Pull Test
Verify successful publication by pulling the mirrored release image directly from your private Harbor registry:

```
docker pull [[harbor-url]]/ocp/openshift/release-images:4.20.14-x86_64
```

