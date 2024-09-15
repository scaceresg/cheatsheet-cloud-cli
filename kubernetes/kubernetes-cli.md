# Kubernetes

Source: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-

- [Kubernetes](#kubernetes)
- [Config](#config)
- [Create, Run, Apply, Delete, Scale](#create-run-apply-delete-scale)
- [Get](#get)
- [Set](#set)
- [Debug: Describe and Logs](#debug-describe-and-logs)

---


# Config

* `cat ~/.kube/config`: Get configuration file (Usually in home `~` 
directory)

* `kubectl config get-users`: Display users defined in the kubeconfig

* `kubectl config get-contexts`: Display one or many contexts from the 
kubeconfig file

* `kubectl config get-clusters`: Display clusters defined in the kubeconfig

* `kubectl config set [property.name] [value]`: Set an individual value in 
a kubeconfig file. 

  For example:

  - Set the cluster field in the "my-context" context to "my-cluster":
  
    `kubectl config set contexts.my-context.cluster my-cluster`

* `kubectl config delete-user [user_name]`: Delete the specified user from 
the kubeconfig

* `kubectl config delete-context [context_name]` : Delete the specified 
context from the kubeconfig

* `kubectl config delete-cluster [cluster_name]`: Delete the specified cluster 
from the kubeconfig

* `kubectl config view`: Display merged kubeconfig settings or a specified 
kubeconfig file. 

  Use flags:

  - `--flatten`: Flatten the resulting kubeconfig file into self-contained 
  output (useful for creating portable kubeconfig files)

  - `--merge`: Merge the full hierarchy of kubeconfig files
  
  - `--minify`: Remove all information not used by current-context from 
  the output

    For example:

    * Display resources of current context by namespace value:

      `kubectl view --minify | grep namespace`

  - `--output` or `-o`: Output format. One of: json|yaml|name|go-template
  |go-template-file|template|templatefile|jsonpath|jsonpath-as-json
  |jsonpath-file.

    For example: 
    
    * Get the password for "user1" user: 
      
      `kubectl config view -o jsonpath='{.users[?(@.name == "user1")].user.password}'`

* `kubectl config current-context`: Display the current-context

* `kubectl config use-context [context_name]`: Use a specific context 
for the cluster

* `kubectl config set-context [context_name] --user=[user_name]`: Set a 
context entry in kubeconfig. 

  Use flags:

  - `--current`: Modify the current context
  
    For example:

    * Set the namespace for the current context to `my-namespace`:

      `kubectl config set-context --current --namespace=my-namespace`
  
  - `--cluster`: Set a cluster 
  
  - `--user`: Set an user
  
  -  `--namespace`: Set a namespace

# Create, Run, Apply, Delete, Scale

* `kubectl create -f [resource_filename]`: Create a resource from a 
file or from stdin. JSON and YAML formats are accepted

  For example:

  - Create a Pod from a manifest file: 

    `kubectl create -f pod-setup.yml`

  Use flags:

  - `--dry-run`: Must be "none", "server", or "client". If client 
  strategy, only print the object that would be sent, without sending 
  it. If server strategy, submit server-side request without persisting 
  the resource

  For example:

    * Save a manifest file for an `nginx` deployment from image:
      
      ```
      kubectl create deployment ngdep --image=nginx \
        --dry-run=client -o yaml > ngdeployment.yaml
      ```

* `kubectl run [pod_name] --image=[image_name]`: Create and run a 
particular image in a pod
  
  For example:

  - Start a nginx pod from image

    `kubectl run nginx --image=nginx`

  Use flags:

  - `--image`: The image for the container to run
  
  - `--port`: The port that this container exposes
  
  - `--env`: Environment variables to set in the container

    For example:

    * Start a hazelcast pod, set port 5701 and environment variable 
    "DNS_DOMAIN=cluster" in the container:

      ```
      kubectl run hazelcast --image=hazelcast/hazelcast --port=5701 \
      --env="DNS_DOMAIN=cluster"
      ```
  - `--dry-run`: Must be "none", "server", or "client". If client 
  strategy, only print the object that would be sent, without sending 
  it. If server strategy, submit server-side request without persisting 
  the resource

    For example:

    * Save the manifest file for running an `nginx` image in a pod 
    in YAML format:

      ```
      kubectl run nginx-pod --image=nginx --dry-run=client -o yaml > ngpod.yaml
      ```

  - `--command`: If true and extra arguments are present, use them as 
  the 'command' field in the container, rather than the 'args' field 
  which is the default
  
  - `--labels` or `-l`: Comma separated labels to apply to the pod(s). Will 
  override previous values

  - `--quiet` or `-q`: If true, suppress prompt messages
  
  - `--restart`: The restart policy for this Pod. Legal values [Always, 
  OnFailure, Never] 

  - `--save-config`: If true, the configuration of current object will be 
  saved in its annotation. Otherwise, the annotation will be unchanged. This 
  flag is useful when you want to perform kubectl apply on this object in 
  the future

* `kubectl apply -f [filename]`: Apply a configuration to a resource by file 
name or stdin. The resource name must be specified. JSON and YAML formats are 
accepted

  Use flags:

  - `--all`: Select all resources in the namespace of the specified resource 
  types

  - `--cascade`: Must be "background", "orphan", or "foreground". 
  Selects the deletion cascading strategy for the dependents (e.g. Pods 
  created by a ReplicationController). Defaults to background

  - `--filename` or `-f`: Containing the configuration to apply
  
  - `--force`: If true, immediately remove resources from API and bypass 
  graceful deletion

  - `--grace-period`: Period of time in seconds given to the resource to 
  terminate gracefully. Ignored if negative. Set to 1 for immediate shutdown. 
  Can only be set to 0 when `--force` is true (force deletion)

* `kubectl delete -f [filename]`: Delete resources by file names, stdin, 
resources and names, or by resources and label selector. JSON and YAML formats 
are accepted

  For example:

  - Delete resources with the namespace `kubekart`:
    
    `kubectl delete ns kubekart`

  - Delete a pod using the type and name specified in `pod.json`:

    `kubectl delete -f ./pod.json`

  - Delete pods and services with label `name=myLabel`:

    `kubectl delete pods,services -l name=myLabel`

  Use flags:

  - `--all`: Delete all resources, including uninitialized ones, in the 
  namespace of the specified resource types

  - `--all-namespaces` or `-A`: If present, list the requested object(s) 
  across all namespaces

  - `--cascade`: Must be "background", "orphan", or "foreground". 
  Selects the deletion cascading strategy for the dependents (e.g. Pods 
  created by a ReplicationController). Defaults to background

  - `--filename` or `-f`: Containing the resource to delete
  
  - `--force`: If true, immediately remove resources from API and bypass 
  graceful deletion 

    For example:

    - Force delete a pod on a dead node: `kubectl delete pod foo --force`
  
  - `--grace-period`: Period of time in seconds given to the resource to 
  terminate gracefully. Ignored if negative. Set to 1 for immediate shutdown. 
  Can only be set to 0 when `--force` is true (force deletion)

  - `--now`: If true, resources are signaled for immediate shutdown (same 
  as --grace-period=1)

  - `--selector` or `-l`: Selector (label query) to filter on, not including 
  uninitialized ones

* `kubectl scale -f [filename]`: Set a new size for a deployment, replica 
set, replication controller, or stateful set

  For example:

  - Scale down a ReplicaSet (`rs`) to 1 replica:

    `kubectl scale --replicas=1 rs/frontend`

  - If the deployment named mysql's current size is 2, scale mysql to 3:

    `kubectl scale --current-replicas=2 --replicas=3 deployment/mysql`
  
  Use flags:

  - `--all`: Select all resources in the namespace of the specified resource 
  types

  - `--current-replicas`: Precondition for current size. Requires that the 
  current size of the resource match this value in order to scale

  - `--filename` or `-f`: Filename, directory, or URL to files identifying the 
  resource to set a new size

  - `--replicas`: The new desired number of replicas. Required
  
  - `--resource-version`: Precondition for resource version. Requires that 
  the current resource version match this value in order to scale

* `kubectl exec --stdin --tty [pod_name] -- /bin/bash` or `-- /bin/sh`:
Log into a Pod 

# Get

* `kubectl get [resource]`: Prints a table of the most important 
information about the specified resources. 

  For example: 

  - Show all the namespaces:
  
    `kubectl get namespaces` or `kubectl get ns`

  - Show all the resources from all the namespaces:

    `kubectl get all --all-namespaces`

  - Show all the services for a specific from the namespace `kube-system`:
  
    `kubectl get services -n kube-system` or `kubectl get svc -n kube-system`
  
  Use flags:

  - `--all-namespaces` or `-a`: List the requested object(s) across all 
  namespaces.

  - `--output wide` or `-o wide`: List the requested object(s) in ps 
  output format with more information. Other formats are YAML or JSON

    For example:

    - Show pod information in YAML format: `kubectl get pod/nginx -o yaml`

  - `--filename` or `-f`: Filename, directory, or URL to files identifying 
  the resource to get from a server

  - `--label-columns` or `-L`: Accepts a comma separated list of labels that 
  are going to be presented as columns. Names are case-sensitive

  - `--sort-by`: Sort list types using this field specification. The field 
  specification is expressed as a JSONPath expression (e.g. 
  `'{.metadata.name}'`)

  - `--show-kind`: List the resource type for the requested object(s)
  
  - `--selector` or `-l`: Selector (label query) to filter on. Supports '=', 
  '==', and '!='.(e.g. `-l key1=value1,key2=value2`)

* `kubectl api-resources`: Print the supported API resources on the server. 

  Use flags:

  - `--namespaced`: If false, non-namespaced resources will be returned, otherwise 
  returning namespaced resources by default

  - `--output wide` or `-o wide`: Print the supported API resources with 
  more information

  - `--sort-by`: If non-empty, sort list of resources using specified field. 
  The field can be either 'name' or 'kind'

# Set

* `kubectl set [subcommand]`: Configure application resources

* `kubectl set image [type_name] [container_name]`: Update existing container 
image(s) of resources

  For example:

  - Set a deployment's nginx container image to 'nginx:1.9.1':

    `kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1`

* `kubectl set env [pod/resource_name] `: Update/List environment variables 
on a pod/resource template

For example:

- Update deployment 'registry' with a new environment variable:
  
  `kubectl set env deployment/registry STORAGE_DIR=/local`

- List the environment variables defined on a deployments 'sample-build':

  `kubectl set env deployment/sample-build --list`

- List the environment variables defined on all pods:

  `kubectl set env pods --all --list`

- Output modified deployment in YAML, and does not alter the object on the 
server:

  `kubectl set env deployment/sample-build STORAGE_DIR=/data -o yaml`

- Update all containers in all replication controllers in the project to 
have ENV=prod:

  `kubectl set env rc --all ENV=prod`

- Import specific keys from a config map:

  `kubectl set env --keys=my-key --from=configmap/myconfigmap deployment/myapp`

* `kubectl set resources [resource_name]`: Specify compute resource 
requirements (CPU, memory) for any resource that defines a pod template

  For example:

  - Set a deployments nginx container cpu limits to "200m" and memory to 
  "512Mi":

    `kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi`
  
  - Remove the resource requests for resources on containers in nginx:
    
    `kubectl set resources deployment nginx --limits=cpu=0,memory=0 --requests=cpu=0,memory=0`

# Debug: Describe and Logs

* `kubectl describe [resource_name/type]`: Show details of a specific resource 
or group of resources, such as IP address, container image, exposed ports, 
events, etc.

  For example:

  - Describe a pod: `kubectl describe pods/nginx`
  
  - Describe all pods: `kubectl describe pods`
  
  - Describe a pod identified by type and name in "pod.json":
  
    `kubectl describe -f pod.json` 

  Use flags:

  - `--all-namespaces` or `-A`: List the requested object(s) across all 
  namespaces.

  - `--filename` or `-f`: Filename, directory, or URL to files containing 
  the resource to describe

  - `--show-events`: If true, display events related to the described object

  - `--selector` or `l`: Selector (label query) to filter on, supports '=', 
  '==', and '!='.(e.g. -l key1=value1,key2=value2)

* `kubectl logs [pod_name]`: Print the logs for a container in a pod or 
specified resource. If the pod has only one container, the container name 
is optional

  For example:

  - Return snapshot logs from pod nginx with only one container: 
    
    `kubectl logs nginx`
  
  - Return snapshot logs from pod nginx with multi containers:

    `kubectl logs nginx --all-containers=true`

  - Display only the most recent 20 lines of output in pod nginx:
  
    `kubectl logs --tail=20 nginx`

  Use flags:

  - `--all-containers`: Get all containers' logs in the pod(s)
  
  - `--container` or `-c`: Print the logs of this container
  
  - `--previous` or `-p`: If true, print the logs for the previous 
  instance of the container in a pod if it exists 

  - `--since`: Only return logs newer than a relative duration like 5s, 
  2m, or 3h. Defaults to all logs

  - `--since-time`: Only return logs after a specific date (RFC3339). 
  Defaults to all logs

  - `--selector` or `-l`: Selector (label query) to filter on

  - `--tail`: Lines of recent log file to display. Defaults to -1 with 
  no selector, showing all log lines otherwise 10, if a selector is 
  provided

  - `--timestamps`: Include timestamps on each line in the log output