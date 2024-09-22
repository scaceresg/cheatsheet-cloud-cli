# Google Kubernetes Engine

**Contents**:

- [Google Kubernetes Engine](#google-kubernetes-engine)
  - [Control Access to GKE Clusters using IAM](#control-access-to-gke-clusters-using-iam)
  - [Create an Autopilot Cluster](#create-an-autopilot-cluster)
  - [Create a Standard Cluster](#create-a-standard-cluster)
  - [Create a Deployment](#create-a-deployment)
  - [Scale Up/Down a Deployment](#scale-updown-a-deployment)
  - [Trigger Deployments Rollouts](#trigger-deployments-rollouts)
  - [Trigger Deployments Rollbacks](#trigger-deployments-rollbacks)
  - [Create a LoadBalancer Service](#create-a-loadbalancer-service)
  - [Perform a Canary Deployment](#perform-a-canary-deployment)
  - [Create a Standard Cluster Network Policy](#create-a-standard-cluster-network-policy)
  - [Restrict Incoming Traffic to Pods](#restrict-incoming-traffic-to-pods)
  - [Restrict Outgoing Traffic from the Pods](#restrict-outgoing-traffic-from-the-pods)
  - [Create PersistentVolume and PersistentVolumeClaim](#create-persistentvolume-and-persistentvolumeclaim)
  - [Create StatefulSets](#create-statefulsets)
  - [Define and Use Pod Security Admission](#define-and-use-pod-security-admission)
  - [Configure a Monitoring Workspace for GKE](#configure-a-monitoring-workspace-for-gke)
    - [Create a Custom Monitoring Dashboard](#create-a-custom-monitoring-dashboard)
    - [Create Alerts with Monitoring](#create-alerts-with-monitoring)

---

## Control Access to GKE Clusters using IAM

Allow a user with **USERNAME** to create a GKE cluster and deploy 
workloads by **granting permissions to administer all GKE clusters 
and manage resources** inside those clusters in this project.

In Cloud Console:

* Go to IAM & admin > IAM

* Locate the row that corresponds to **USERNAME**, and then click 
on the pencil icon at the right-end of that row to edit that user's 
permissions.

* Click ADD ANOTHER ROLE to add:
  
  - Kubernetes Engine > Kubernetes Engine Cluster Admin

**USERNAME** still lacks some of the rights necessary to deploy a 
cluster. This is because GKE leverages Google Cloud Compute Engine 
instances for the nodes.

**USERNAME** must also be assigned the `iam.serviceAccountUser` 
role on the Compute Engine default service account.

* In Cloud Console, go to IAM & admin > Service accounts

* In the IAM console, click the row that corresponds to the Compute 
Engine default service account to select it.

* Click on the Permissions tab > Grant access

* In the permissions information panel:

  - Type the username for **USERNAME** into the New principals box

  - In the Select a role box, make sure Service Accounts > Service 
  Account User is selected

  - Save

## Create an Autopilot Cluster

In Cloud Shell:

In Cloud Shell:

* Check the active account name:
  
  ```
  gcloud auth list
  ```

* List the Project ID:
  
  ```
  gcloud config list project
  ```

* Set the environment variable for the zone and cluster name:

  ```
  $ export my_region=us-west1
  $ export my_cluster=autopilot-cluster-1
  ```

* Configure `kubectl` tab completion in Cloud Shell:
  
  ```
  source <(kubectl completion bash)
  ```

* Create an Autopilot cluster:
  
  ```
  gcloud container clusters create-auto $my_cluster --region $my_region
  ```

* Configure access to your cluster for the `kubectl` command-line 
tool:

  ```
  gcloud container clusters get-credentials $my_cluster 
    --region $my_region
  ```

## Create a Standard Cluster

In Cloud Shell:

* Check the active account name:
  
  ```
  gcloud auth list
  ```

* List the Project ID:
  
  ```
  gcloud config list project
  ```

* Set the environment variable for the zone and cluster name:

  ```
  $ export [my_zone=us-west1-a | $my_region=us-west1]
  $ export my_cluster=standard-cluster-1
  ```

* Configure `kubectl` tab completion in Cloud Shell:
  
  ```
  source <(kubectl completion bash)
  ```

* Create a Kubernetes Standard cluster:

  ```
  gcloud container clusters create $my_cluster \
    --num-nodes 3 [--zone $my_zone | --region $my_region]
  ```

* Configure access to your cluster for the `kubectl` command-line 
tool:

  ```
  gcloud container clusters get-credentials $my_cluster 
    [--zone $my_zone | --region $my_region]
  ```

## Create a Deployment

This Deployment is configured to run three Pod replicas with a 
single `nginx` container in each Pod listening on TCP port 80.

* `nginx-deployment.yaml`
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels:
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.7.9
          ports:
          - containerPort: 80
  ```

* Create a Deployment using a sample deployment manifest called 
`nginx-deployment.yaml`:

  ```
  kubectl apply -f ./nginx-deployment.yaml
  ```

* View the list of Deployments:
  
  ```
  kubectl get deployments
  ```

* View detailed information about the Deployment:
  
  ```
  gcloud container clusters describe $my_cluster \
    [--zone $my_zone | --region $my_region]
  ```

## Scale Up/Down a Deployment

* Scale down the `nginx-deployment` to 1 Pod replicas:

  ```
  kubectl scale --replicas=1 deployment nginx-deployment
  ```

## Trigger Deployments Rollouts

A deployment's rollout is triggered if and only if the Deployment's 
Pod template (i.e., `.spec.template`) is changed, e.g., if the 
labels or container images of the template are updated.

* Trigger a deployment rollout by updating the version of `nginx` 
in the deployment:

  ```
  kubectl set image deployment.v1.apps/nginx-deployment
    nginx=nginx:1.9.1 
  ```

* Annotate the rollout with details on the change:
  
  ```
  kubectl annotate deployment nginx-deployment \
    kubernetes.io/change-cause="version change to 1.9.1" \
    --overwrite=true
  ```

* View rollout status:
  
  ```
  kubectl rollout status deployment.v1.apps/nginx-deployment
  ```

* View the rollout history of the deployment:
  
  ```
  kubectl rollout history deployment nginx-deployment
  ```

## Trigger Deployments Rollbacks

* Roll back to the previous version of the `nginx` deployment:

  ```
  kubectl rollout undo deployments nginx-deployment
  ```

* View the updated rollout history of the deployment:
  
  ```
  kubectl rollout history deployment nginx-deployment
  ```

* View the details of the latest deployment revision:
  
  ```
  kubectl rollout history deployment/nginx-deployment \
    --revision=3
  ```

## Create a LoadBalancer Service

Services control inbound traffic to an application. The 
LoadBalancer service is configured to distribute inbound traffic 
on TCP port 60000 to port 80 on any containers that have the 
label `app: nginx`.

* `service-nginx.yaml`:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx
  spec:
    type: LoadBalancer
    selector:
      app: nginx
    ports:
    - protocol: TCP
      port: 60000
      targetPort: 80
  ```

* Deploy a `LoadBalancer` service type using a manifest file called 
`service-nginx.yaml`:

  ```
  kubectl apply -f ./service-nginx.yaml
  ```

* View the details of the nginx service:
  
  ```
  kubectl get service nginx
  ```

## Perform a Canary Deployment

A Canary Deployment is a separate Deployment used to test a new 
version of your application. A single service targets both the 
canary and normal deployments. And it can direct a subset of users 
to the canary version to mitigate the risk of new releases.

* `nginx-canary.yaml`:
  
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-canary
    labels:
      app: nginx
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
          track: canary
          Version: 1.9.1
      spec:
        containers:
        - name: nginx
          image: nginx:1.9.1
          ports:
          - containerPort: 80
  ```

The manifest for the nginx Service you deployed in the previous 
task uses a label selector to target the Pods with the `app: nginx` 
label. Both the normal Deployment and this new Canary Deployment 
have the `app: nginx` label.

Inbound connections will be distributed by the Service to both the 
normal and canary deployment Pods. The Canary Deployment has fewer 
replicas (Pods) than the normal Deployment, and thus it is available 
to fewer users than the normal deployment.

* Create the canary deployment based on the configuration file:
  
  ```
  kubectl apply -f ./nginx-canary.yaml
  ```

* Verify that both the nginx and the `nginx-canary` deployments are 
present:

  ```
  kubectl get deployments
  ```

## Create a Standard Cluster Network Policy

* Create a cluster network policy to restrict communication between
the Pods. A zero-trust zone is important to prevent lateral attacks 
within the cluster when an intruder compromises one of the Pods.

  ```
  gcloud container clusters create $my_cluster \
    --num-nodes 3 --enable-ip-alias --zone $my_zone \
    --enable-network-policy
  ```

* Configure access to your cluster for the `kubectl` command-line 
tool:

  ```
  gcloud container clusters get-credentials $my_cluster \
    --zone $my_zone
  ```

* Run a simple web server application with the label `app=hello`, 
and expose the web application internally in the cluster:

  ```
  kubectl run hello-web --labels app=hello \
    --image=gcr.io/google-samples/hello-app:1.0 \
    --port 8080 --expose
  ```

## Restrict Incoming Traffic to Pods

Create a sample NetworkPolicy by defining an ingress policy that 
allows access to Pods labeled `app: hello` from Pods labeled 
`app: foo`

* `hello-allow-from-foo.yaml`:
  
  ```
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: hello-allow-from-foo
  spec:
    policyTypes:
    - Ingress
    podSelector:
      matchLabels:
        app: hello
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: foo
  ```

* Create the ingress policy:
  
  ```
  kubectl apply -f hello-allow-from-foo.yaml
  ```

* Verify that the policy was created:
  
  ```
  kubectl get networkpolicy
  ```
  ```
  NAME                   POD-SELECTOR   AGE
  hello-allow-from-foo   app=hello      7s
  ```

* Validate the ingress policy by running a temporary Pod called 
`test-1` with the label `app=foo` and get a shell in the Pod:

  ```
  kubectl run test-1 --labels app=foo --image=alpine \
    --restart=Never --rm --stdin --tty
  ```
 
  - `--stdin` (alternatively `-i`): creates an interactive session 
  attached to STDIN on the container.

  - `--tty` (alternatively `-t`): allocates a TTY for each container 
  in the pod.

  - `--rm`: instructs Kubernetes to treat this as a temporary Pod 
  that will be removed as soon as it completes its startup task. As 
  this is an interactive session it will be removed as   soon as the 
  user exits the session.

  - `--label` (alternatively `-l`): adds a set of labels to the pod.

  - `--restart`: defines the restart policy for the Pod.
  
* Make a request to the `hello-web:8080` endpoint to verify that the 
incoming traffic is allowed:

  ```
  # wget -qO- --timeout=2 http://hello-web:8080
  ```
  ```
  If you don't see a command prompt, try pressing enter.
  / # wget -qO- --timeout=2 http://hello-web:8080
  Hello, world!
  Version: 1.0.0
  Hostname: hello-web-8b44b849-k96lh
  / #
  ```

* Now, run a different Pod using the same Pod name but using a label, 
`app=other`, that does not match the podSelector in the active 
network policy. This Pod should not have the ability to access the 
`hello-web` application.

  ```
  kubectl run test-1 --labels app=other --image=alpine \
    --restart=Never --rm --stdin --tty
  ```

* Make a request to the hello-web:8080 endpoint to verify that the 
incoming traffic is not allowed:

  ```
  # wget -qO- --timeout=2 http://hello-web:8080
  ```
  ```
  If you don't see a command prompt, try pressing enter.
  / # wget -qO- --timeout=2 http://hello-web:8080
  wget: download timed out
  / #
  ```

## Restrict Outgoing Traffic from the Pods

You can restrict outgoing (egress) traffic as you do incoming 
traffic. However, in order to query internal hostnames (such as 
`hello-web`) or external hostnames (such as `www.example.com`), you 
must allow DNS resolution in your egress network policies. DNS 
traffic occurs on port 53, using TCP and UDP protocols.

Create a NetworkPolicy manifest file `foo-allow-to-hello.yaml`. This 
file will define:

* A policy that permits Pods with the label `app: foo` to communicate 
with Pods labeled `app: hello` on any port number, and

* Also allows the Pods labeled `app: foo` to communicate to any 
computer on UDP port 53, which is used for DNS resolution. Without 
the DNS port open, you will not be able to resolve the hostnames.

* `foo-allow-to-hello.yaml`:

  ```
  kind: NetworkPolicy
  apiVersion: networking.k8s.io/v1
  metadata:
    name: foo-allow-to-hello
  spec:
    policyTypes:
    - Egress
    podSelector:
      matchLabels:
        app: foo
    egress:
    - to:
      - podSelector:
          matchLabels:
            app: hello
    - to:
      ports:
      - protocol: UDP
        port: 53
  ```

* Create the egress policy:
  
  ```
  kubectl apply -f foo-allow-to-hello.yaml
  ```

* Verify that the policy was created:
  
  ```
  kubectl get networkpolicy
  ```
  ```
  NAME                   POD-SELECTOR   AGE
  foo-allow-to-hello     app=foo        7s
  hello-allow-from-foo   app=hello      5m
  ```

* Validate the egress policy by running a new web application called 
`hello-web-2` and expose it internally in the cluster:
  
  ```
  kubectl run hello-web-2 --labels app=hello-2 \
    --image=gcr.io/google-samples/hello-app:1.0 \
    --port 8080 --expose
  ```

* Again, create a temporary Pod with label `app=foo`:
  
  - Test connection to http://hello-web:8080 $\rightarrow$ CAN 
  establish connection because of label `app: hello`

  - Test connection to http://hello-web-2:8080 $\rightarrow$ CANNOT 
  establish connection because of label `app=hello-2`

  - Test connection to http://www.example.com $\rightarrow$ CANNOT 
  establish connection because the network policies do not allow 
  external http traffic (tcp port 80)
  
## Create PersistentVolume and PersistentVolumeClaim

You don't need to directly configure PV objects or create Compute 
Engine persistent disks. Instead, you can create a PVC, and 
Kubernetes automatically provisions a persistent disk for you.

* Create a 30 gigabyte PVC called `hello-web-disk` that can be 
mounted as a read-write volume on a single node at a time.

* `pvc-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: hello-web-disk
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 30Gi
  ```

* Show that you currently have no PVCs:
  
  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  No resources found in default namespace.
  ```

* Create the PVC:
  
  ```
  kubectl apply -f pvc-demo.yaml
  ```

* Show list of PVCs:
  
  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGE CLASS   AGE
  hello-web-disk Pending                                          standard-rwo    15s
  ```

Mount the PVC to a Pod and attach the `pvc-demo-volume` to the Pod 
and mount that volume to the path `/var/www/html` inside the nginx 
container.

Files saved to this directory inside the container will be saved to 
the persistent volume and persist even if the Pod and the container 
are shutdown and recreated.

* `pod-volume-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: pvc-demo-pod
  spec:
    containers:
      - name: frontend
        image: nginx
        volumeMounts:
        - mountPath: "/var/www/html"
          name: pvc-demo-volume
    volumes:
      - name: pvc-demo-volume
        persistentVolumeClaim:
          claimName: hello-web-disk
  ```

* Create the Pod with the volume:
  
  ```
  kubectl apply -f pod-volume-demo.yaml
  ```

* List the Pods in the cluster:
  
  ```
  kubectl get pods
  ```
  
  - The volume is mounted before the status changes to "Running".

* Verify the PVC is accessible within the Pod, you must gain shell 
access to your Pod. To start the shell session, execute the following 
command:

  ```
  kubectl exec -it pvc-demo-pod -- sh
  ```

* Create a simple text message as a web page in the Pod enter the 
following commands:

  ```
  # echo Test webpage in a persistent volume!>/var/www/html/index.html
  # chmod +x /var/www/html/index.html
  ```

* Verify the text file contains your message and then exit:
  
  ```
  # cat /var/www/html/index.html
  ```
  ```
  Test webpage in a persistent volume!
  ```

* Test the persistency of the PVC by deleting the Pod. Show your PVC:

  ```
  kubectl get persistentvolumeclaim
  ```
  ```
  NAME           STATUS    VOLUME       CAPACITY   ACCESS MODES   STORAGE CLASS   AGE
  hello-web-disk Bound     pvc-8...34   30Gi       RWO            standard-rwo    22m
  ```

* Redeploy and access the Pod again to verify that the text file 
still contains your message.

## Create StatefulSets

A StatefulSet is like a Deployment, except that the Pods are given 
unique identifiers.

Use your PVC from the previous section to create a StatefulSet. 
Before you can use the PVC with the StatefulSet, you must delete 
the Pod that is currently using it.

* Create a manifest file `statefulset-demo.yaml` that creates a 
StatefulSet:

  - The manifest includes a LoadBalancer Service and 3 replicas of 
  a Pod containing an `nginx` container 
  
  - It also contains a `volumeClaimTemplate` for 30 gigabyte PVCs 
  with the name `hello-web-disk`. 
  
  - The nginx containers mount the PVC called `hello-web-disk` at 
  `/var/www/html`:

* `statefulset-demo.yaml`:
  
  ```
  apiVersion: v1
  kind: Service
  metadata:
    name: statefulset-demo-service
  spec:
    ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
    type: LoadBalancer
  ---

  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: statefulset-demo
  spec:
    selector:
      matchLabels:
        app: MyApp
    serviceName: statefulset-demo-service
    replicas: 3
    updateStrategy:
      type: RollingUpdate
    template:
      metadata:
        labels:
          app: MyApp
      spec:
        containers:
        - name: stateful-set-container
          image: nginx
          ports:
          - containerPort: 80
            name: http
          volumeMounts:
          - name: hello-web-disk
            mountPath: "/var/www/html"
    volumeClaimTemplates:
    - metadata:
        name: hello-web-disk
      spec:
        accessModes: [ "ReadWriteOnce" ]
        resources:
          requests:
            storage: 30Gi
  ```

* Create the StatefulSet with the volume:
  
  ```
  kubectl apply -f statefulset-demo.yaml
  ```
  ```
  service "statefulset-demo-service" created
  statefulset.apps "statefulset-demo" created
  ```

You now have a StatefulSet running behind a service named 
`statefulset-demo-service`!

* View the StatefulSet details:
  
  ```
  kubectl describe statefulset statefulset-demo
  ```
  ```
  Normal  SuccessfulCreate  10s   statefulset-controller
  Message: create Claim hello-web-disk-statefulset-demo-0 Pod statefulset-demo-0 in StatefulSet statefulset-demo success

  Normal  SuccessfulCreate  10s   statefulset-controller
  Message: create Pod statefulset-demo-0 in StatefulSet statefulset-demo successful
  ```

* List the Pods in the cluster:
  
  ```
  kubectl get pods
  ```
  ```
  NAME                 READY     STATUS    RESTARTS   AGE
  statefulset-demo-0   1/1       Running   0          6m
  statefulset-demo-1   1/1       Running   0          3m
  statefulset-demo-2   1/1       Running   0          2m
  ```

* List the PVCs:
  
  ```
  kubectl get pvc
  ```
  ```
  NAME                          STATUS    VOLUME           CAPACITY ACCESS
  hello-web-disk                Bound     pvc-86117ea6-...34   30Gi    RWO
  hello-web-disk-st...-demo-0   Bound     pvc-92d21d0f-...34   30Gi    RWO
  hello-web-disk-st...-demo-1   Bound     pvc-9bc84d92-...34   30Gi    RWO
  hello-web-disk-st...-demo-2   Bound     pvc-a526ecdf-...34   30Gi    RWO
  ```

The original hello-web-disk is still there and you can now see the 
individual PVCs that were created for each Pod in the new 
statefulset Pod.

* View the details of the first PVC in the StatefulSet:
  
  ```
  kubectl describe pvc hello-web-disk-statefulset-demo-0
  ```

* Verify the connection of Pods in StatefulSets to particular PVs 
as the Pods are stopped and restarted:

  - Gain shell access to your Pod to start the shell session
    
    ```
    kubectl exec -it statefulset-demo-0 -- sh
    ```
  
  - Verify that there is no `index.html` text file in the 
  `/var/www/html` directory

  - Create a simple text message as a web page in the Pod:
    
    ```
    # echo Test webpage in a persistent volume!>/var/www/html/index.html
    # chmod +x /var/www/html/index.html
    ```

  - Delete the Pod where you updated the file on the PVC:
    
    ```
    kubectl delete pod statefulset-demo-0
    ```

  - You will see that the StatefulSet is automatically restarting 
  the `statefulset-demo-0` Pod. Connect again to the shell on the 
  new `statefulset-demo-0` Pod and verify the `index.html` file 
  still contains your message.

## Define and Use Pod Security Admission

PodSecurity is a Kubernetes admission controller that lets you 
apply **Pod Security Standards** to Pods running on your GKE 
clusters. Pod Security Standards are **predefined security 
policies that cover the high-level needs of Pod security** in 
Kubernetes. These policies range from being highly permissive to 
highly restrictive.

Create a Pod Security Policy to create **unprivileged Pods for
specific namespaces** in the cluster:

* Unprivileged Pods do not allow users to execute code as root, 
and have limited access to devices on the host.

* Create environment variables for the Google Cloud zone and 
cluster name:

  ```
  $ export my_zone=ZONE
  $ export my_cluster=standard-cluster-1
  ```

* Configure tab completion for the kubectl command-line tool:
  
  ```
  source <(kubectl completion bash)
  ```

* Configure access to your cluster for kubectl:
  
  ```
  gcloud container clusters get-credentials $my_cluster \
    --zone $my_zone
  ```

* Create namespaces in your cluster:

  - `baseline-ns`: For permissive workloads
  
  - `restricted-ns`: For highly restricted workloads
  
  ```
  $ kubectl create ns baseline-ns
  $ kubectl create ns restricted-ns
  ```

* Apply the following Pod Security Standards:
  
  - Workloads in the `baseline-ns` namespace that violate the 
  baseline policy are allowed, and the client displays a warning 
  message:
  
    * **baseline**: Apply to `baseline-ns` in the **warn** mode
  
  - Workloads in the `restricted-ns` namespace that violate the 
  restricted policy are rejected, and GKE adds an entry to the 
  audit logs:

    * **restricted**: Apply to `restricted-ns` in the **enforce** 
    mode
  
  ```
  $ kubectl label --overwrite ns baseline-ns \
    pod-security.kubernetes.io/warn=baseline
  $ kubectl label --overwrite ns restricted-ns \
    pod-security.kubernetes.io/enforce=restricted
  ```

* Verify that the labels were added:
  
  ```
  kubectl get ns --show-labels
  ```
  ```
  baseline-ns       Active   74s   kubernetes.io/metadata.name=baseline-ns,pod-security.kubernetes.io/warn=baseline
  restricted-ns     Active   18s   kubernetes.io/metadata.name=restricted-ns,pod-security.kubernetes.io/enforce=restricted
  default           Active   57m   kubernetes.io/metadata.name=default
  kube-public       Active   57m   kubernetes.io/metadata.name=kube-public
  kube-system       Active   57m   kubernetes.io/metadata.name=kube-system
  ```

Verify the PodSecurity admission controller is working by 
deploying a Pod running a container with priviliged escalation 
`securityContext: privileged: true` to both namespaces 
`baseline-ns` and `restricted-ns`:

* `psa-workload.yaml`:
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels:
      app: nginx
  spec:
    containers:
    - name: nginx
      image: nginx
      securityContext:
        privileged: true
  ```

* Apply to the `baseline-ns` namespace. It's created with a warning 
message:
  
  ```
  kubectl apply -f psa-workload.yaml --namespace=baseline-ns
  ```
  ```
  Warning: would violate PodSecurity "baseline:latest": privileged (container "nginx" must not set securityContext.privileged=true)
  pod/nginx created
  ```

* Apply to the `restricted-ns` namespace: Pod won't be created:
  
  ```
  kubectl apply -f psa-workload.yaml --namespace=restricted-ns
  ```
  ```
  Error from server (Forbidden): error when creating "workload.yaml": pods "nginx"
  is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation
  != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false),
  unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]),
  runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true),
  seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type
  to "RuntimeDefault" or "Localhost")
  ```

Create a ClusterRole that can be used in a ClusterRoleBinding to 
**tie the policy to accounts** that require to deploy pods with 
unprivileged access:
  
  - Used in ClusterRolebinding that ties the policy to accounts 
  that require the ability to deploy pods with unprivileged access.

## Configure a Monitoring Workspace for GKE

In Cloud Console:

* Click on Navigation menu > **Monitoring**

When the Monitoring dashboard opens, browse the 3 sections of the
Kubernetes Engine Monitoring interface (Refresh if needed):

* Infrastructure

* Workloads

* Services

In the Monitoring interface:

* Click GKE in the **Dashboards** section to view the new 
monitoring interface

* The dashboard shows the health of your GKE clusters and their
workloads

* A dynamic **Timeline** is displayed in the top portion of the 
interface. If required, expand it by clicking on the dropdown icon. 
You can adjust the Time Span from the top of the screen: 1h, 6h, 
1d, 1w, 1m, 6w, or custom 

* This timeline will include markers that indicate **the occurrence 
of alerts (also known as incidents)**

* Click the **Auto refresh button** to allow the screen to update 
as new events are received

* The lower portion of this dashboard contains a multiple section 
views of your clusters and their workloads. The interface is divided 
into several sections: 

  - **Clusters, Nodes and Pods** section: 
    
    Allow you to check the health of particular elements in the 
    cluster. You can also use this to inspect the pods which are 
    running on a particular node in the cluster
  
  - **Workloads** section:
    
    Used to look for workloads which do not have services exposed
  
  - **Kubernetes services** section:
    
    Organises the services configured in your environment **by 
    cluster**, then **by namespace** (the administrative barrier or 
    partition within the cluster), and then shows the various 
    services available to users within that namespace.

  - **Namespaces** section:
    
    Shows the list of namespaces within the cluster

You can click on any of the section elements and then on "View all"
and **Metrics** to see more metrics. You can also click the 
**Logs** tab to view the log activity for that element.

### Create a Custom Monitoring Dashboard

You can create custom dashboards to display important metrics such
as CPU utilisation, container restarts and more custom metrics.

* Go to the Monitoring page at the left-hand side and click on 
**Metrics Explorer**

* Click on Select a metric

* Search the metric: e.g. **Kubernetes Container > Popular Metrics >
CPU request utilization**

* Click Apply

* Click the **Save Chart** button in the upper right corner of the
screen

* Give the chart a name: e.g., **Container CPU Request** 

* Click on **Dashboard > New Dashboard** and name your deashboard:
e.g., **Container Dashboard**

* Click on Save Chart

* Now, launch your dashboard by clicking **Dashboards** in the
navigation pane and then select the name of your new dashboard

* You can add more charts for other metrics:
  
  - Again, go to Metrics Explorer > Select a metric: e.g., 
  **Kubernetes Pod > Custom metrics > Web App - Active Users**

  - Click Apply
  
  - Click on Save Chart and give a name to the new chart: e.g.,
  **Active Users**

  - Select **Container Dashboard** from the dashboards dropdown
  
  - Click Save Chart

* Go back to your dashboard and click the **Gear** icon to display 
the settings menu:

  - Click **Legends > Table** to display the text under each chart

* You can also filter or even aggregate data in the chart using the 
labels in the vertical bars next to the word **Value** at the right 
of each chart.

### Create Alerts with Monitoring

Create an Alert Policy to detect high CPU utilisation among the 
containers:

In Cloud Console:

* Use the Navigation menu to select **Monitoring > Detect >
Alerting**

* Click **+ Create Policy**

* Click on **Select a metric** dropdown

* Uncheck the Active option

* Type: **Kubernetes Container** in filter by resource and metric
name

* Click on **Kubernetes Container > Container > CPU request 
utilization**

* Click Apply

* Set **Rolling windows** to 1 min and click Next

* Set Threshold position to **Above Threshold** and **Threshold
value** to 0.99

* Click Next

**Configure notifications**:

* Click on dropdown arrow next to **Notification Channels**, then
click on **Manage Notification Channels** and finally open the
**Notification channels** page.

* Select the preferred notification channel and click on **ADD 
NEW**

* Save and go back to the previous **Create alerting policy** tab

* Click on Notification Channels again, then click on the Refresh
icon to get your newly added notification channel.

* Click OK, name the alert and click Next

* Review and click **Create Policy**

