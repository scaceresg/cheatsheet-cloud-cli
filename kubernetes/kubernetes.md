# Kubernetes Manifest File Instructions

- [Kubernetes Manifest File Instructions](#kubernetes-manifest-file-instructions)
- [Install Kops and Kubectl](#install-kops-and-kubectl)
- [Deployments](#deployments)
- [Deployment Sequence](#deployment-sequence)
  - [Create and Check Status](#create-and-check-status)
  - [Rollouts and Revision History](#rollouts-and-revision-history)
  - [Rollback to a Previous Revision](#rollback-to-a-previous-revision)
  - [Scale](#scale)
  - [Delete a Deployment](#delete-a-deployment)
- [Mount a Volume to a Pod](#mount-a-volume-to-a-pod)
- [Mount ConfigMap to a Pod](#mount-configmap-to-a-pod)
- [Add Secrets to a Pod](#add-secrets-to-a-pod)
- [Pull an Image from a Private Registry: Docker Hub](#pull-an-image-from-a-private-registry-docker-hub)

---
# Install Kops and Kubectl

* Kops: Download a Kops binary version from: 

  https://github.com/kubernetes/kops/releases, make it 
  executable and move it to the `usr/local/bin/kops` 
  directory with the name `kops`

* Kubectl: Download the binary using the command in:
  
  https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/, 
  make it executable and move it to the `usr/local/bin/` 
  directory

---
# Deployments

* `nginx-deployment.yaml`:

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
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

  Explanation:

  - A Deployment named `nginx-deployment` is created, 
  indicated by the `.metadata.name` field. This name will 
  become the basis for the ReplicaSets and Pods which are 
  created later
    
  - The Deployment creates a ReplicaSet that creates three 
  replicated Pods, indicated by the `.spec.replicas` field

  - The `.spec.selector` field defines how the created 
  ReplicaSet finds which Pods to manage. In this case, you 
  select a label that is defined in the Pod template 
  (`app: nginx`)

  - The `template` field contains the following sub-fields:

    * The Pods are labeled `app: nginx` using the 
    `.metadata.labels` field

    * The Pod template's specification, or `.template.spec` 
    field, indicates that the Pods run one container, `nginx`, 
    which runs the `nginx` Docker Hub image at version 1.14.2

    * Create one container and name it `nginx` using the 
    `.spec.template.spec.containers[0].name` field

# Deployment Sequence

## Create and Check Status

* Create a Deployment from a manifest/definition file in YAML 
format:
  
  ```
  kubectl apply -f [manifest_filename].yaml
  ```

  For example:

  ```
  kubectl apply -f nginx-deployment.yaml
  ```

* Create a Deployment from a manifest/definition file placed 
in a URL:
  
  ```
  kubectl apply -f https://k8s.io/examples/controllers/[manifest_filename].yaml
  ```

* Check list of Deployments: 
  
  ```
  kubectl get deployments
  ```
  ```
  NAME               READY   UP-TO-DATE   AVAILABLE   AGE
  nginx-deployment   3/3     3            3           18s
  ```

* Check rollout status of the Deployment:
  
  ```
  kubectl rollout status deployment/[deploy_name]
  ```

  For example:

  ```
  kubectl rollout status deployment/nginx-deployment
  ```

* Get the ReplicaSet (`rs`) created by the Deployment:
  
  ```
  kubectl get rs
  ``` 
  ```
  NAME                          DESIRED   CURRENT   READY   AGE
  nginx-deployment-75675f5897   3         3         3       18s
  ```

## Rollouts and Revision History

* Update the container image of a specified deployment:
  
  ```
  kubectl set image deployment/[deploy_name] [cont_name]=[image]:[tag]
  ```

  For example:

  - Update the nginx Pods to use the `nginx:1.16.1` image 
  instead of the `nginx:1.14.2` image:

    ```
    kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    ```

* Get Deployment details:
  
  ```
  kubectl describe deployments
  ```
  ```
  Name:                   nginx-deployment
  Namespace:              default
  CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
  Labels:                 app=nginx
  Annotations:            deployment.kubernetes.io/revision=2
  Selector:               app=nginx
  Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
  StrategyType:           RollingUpdate
  MinReadySeconds:        0
  RollingUpdateStrategy:  25% max unavailable, 25% max surge
  Pod Template:
    Labels:  app=nginx
    Containers:
      nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
      Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
      Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
      Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
  ```

* Check rollout history of Deployments:
  
  ```
  kubectl rollout history deployment/[deploy_name]
  ```

  - For the `nginx-deployment` example:
  
  ```
  deployments "nginx-deployment"
  REVISION    CHANGE-CAUSE
  1           kubectl apply --filename=nginx-deployment.yaml
  2           kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
  3           kubectl set image deployment/nginx-deployment nginx=nginx:1.161
  ```

* Show details of each revision: 
  
  ```
  kubectl rollout history deployment/[deploy_name] --revision=[rev_num]
  ``` 

  For example:

  ```
  kubectl rollout history deployment/nginx-deployment --revision=2
  ```

  ```
  deployments "nginx-deployment" revision 2
    Labels:       app=nginx
            pod-template-hash=1159050644
    Annotations:  kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
    Containers:
    nginx:
      Image:      nginx:1.16.1
      Port:       80/TCP
      QoS Tier:
          cpu:      BestEffort
          memory:   BestEffort
      Environment Variables:      <none>
    No volumes.
  ```

* Pause rollouts for that Deployment before you trigger one 
or more updates: 
  
  ```
  kubectl rollout pause deployment/[deploy_name]
  ```

## Rollback to a Previous Revision

* Rollback to the previous revision:
  
  ```
  kubectl rollout undo deployment/[deploy_name]
  ```

* Rollback to a specific revision:
  
  ```
  kubectl rollout undo deployment/[deploy_name] --to-revision=[rev_num]
  ```

  For example:

  ```
  kubectl rollout undo deployment/nginx-deployment --to-revision=2
  ```

## Scale

* Scale a Deployment by setting a new number of replicas:
  
  ```
  kubectl scale deployment/[deploy_name] --replicas=[rep_num]
  ``` 

* Set up an autoscaler for a Deployment and choose the minimum and 
maximum number of Pods to run based on CPU utilization of existing 
Pods:
  
  ```
  kubectl autoscale deployment/[deploy_name] \
    --min=[min_num] --max=[max_num] --cpu-percent=[cpu_usage]
  ```

  For example:

  ```
  kubectl autoscale deployment/nginx-deployment \
    --min=10 --max=15 --cpu-percent=80
  ```

## Delete a Deployment

* Delete Deployment:
  
  ```
  kubectl delete deployment/[deploy_name]
  ```

---
# Mount a Volume to a Pod

Source: https://kubernetes.io/docs/concepts/storage/volumes/

**Steps to**: Create a Pod with a `hostPath` volume that 
mounts a file or directory from the host node's filesystem.

* Manifest to mount `/data/foo` on the host as `/foo` inside
the container running within the `hostpath-example-linux`
Pod:

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: hostpath-example-linux
  spec:
    os: { name: linux }
    nodeSelector:
      kubernetes.io/os: linux
    containers:
    - name: example-container
      image: registry.k8s.io/test-webserver
      volumeMounts:
      - mountPath: /foo
        name: example-volume
        readOnly: true # The mount into the container is read-only
    volumes:
    - name: example-volume
      # mount /data/foo
      hostPath:
        path: /data/foo # directory location on host
        type: Directory # A directory must exist at the given path
  ```

* Change hostPath `type` value: `""`, `"DirectoryOrCreate"`, 
`"Directory"`, `"FileOrCreate"`, `"File"`, `"Socket"`, 
`"CharDevice"` or `"BlockDevice"`

  - 

---
# Mount ConfigMap to a Pod

Source: https://kubernetes.io/docs/concepts/configuration/configmap/

**Steps to**: Create a Pod that uses a ConfigMap object that 
contains keys with multiple values ("file-like keys")

* Create a ConfigMap file: `vim samplecm.yaml`:

  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: game-demo
  data:
    # property-like keys; each key maps to a simple value
    player_initial_lives: "3"
    ui_properties_file_name: "user-interface.properties"

    # file-like keys
    game.properties: |
      enemy.types=aliens,monsters
      player.maximum-lives=5    
    user-interface.properties: |
      color.good=purple
      color.bad=yellow
      allow.textmode=true 
  ```

* Create ConfigMap: 
  
  ```
  kubectl apply -f samplecm.yaml
  ```

* Check the status of the ConfigMap created: 
  
  ```
  kubectl get cm
  ```

* Open the detailed ConfigMap file in YAML: 
  
  ```
  kubectl get cm game-demo -o yaml
  ```

* Check the keys in the `data` section:

  ```
  data:
    game.properties: "enemy.types=aliens,monsters\nplayer.maximum-lives=5  \n"
    player_initial_lives: "3"
    ui_properties_file_name: "user-interface.properties"
    user-interface.properties: "color.good=purple\ncolor.bad=yellow\nallow.textmode=true \n"
  ```

* Create a Pod that has access to a read-only volume where 
the file with the variable keys is located: Pod definition 
file: `readcmpod.yaml`
  
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: configmap-demo-pod
  spec:
    containers:
        - name: demo
        image: alpine
        command: ["sleep", "3600"]
        env:
          # Define the environment variable
          - name: PLAYER_INITIAL_LIVES # Notice that the case is different here
                                       # from the key name in the ConfigMap.
          valueFrom:
            configMapKeyRef:
              name: game-demo           # The ConfigMap this value comes from.
              key: player_initial_lives # The key to fetch.
          - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
        # Mount ConfigMap as a volume
        volumeMounts: 
        - name: config
          mountPath: "/config"
          readOnly: true
  volumes:
    # You set volumes at the Pod level, then mount them into containers 
    # inside that Pod
    - name: config
      configMap:
      # Provide the name of the ConfigMap you want to mount.
        name: game-demo
        # An array of keys from the ConfigMap to create as files
        items:
        - key: "game.properties"
          path: "game.properties"
        - key: "user-interface.properties"
          path: "user-interface.properties"
  ```

* Key-files `"game.properties"` and 
`"user-interface.properties"` are created inside the Pod 
in the `/config` directory

* Create the Pod by running: 
  
  ```
  kubectl apply -f readcmpod.yaml
  ```

* Check the Pod status: 
  
  ```
  kubectl get pod
  ```

* You can log into the Pod to check if the variables were 
mapped correctly:

  ```
  kubectl exec --stdin --tty configmap-demo-pod -- /bin/sh
  ```

  - (`sh` because the Pod is running the `alpine` Image)

  - Check the contents of the `/config` directory: 
  `ls /config/`
  
  - Print the contents of the `.properties` files inside 
  `/config`:
  
    ```
    # cat /config/game.properties
    ``` 
    ```
    enemy.types=aliens,monsters
    player.maximum-lives=5 
    ```
    ```
    # cat /config/user-interface.properties
    ```
    ```
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    ```

  - Check that `echo $PLAYER_INITIAL_LIVES` = `3`

  - Check that `echo $UI_PROPERTIES_FILE_NAME` = 
  `user-interface.properties`
 
---
# Add Secrets to a Pod

Source: https://kubernetes.io/docs/concepts/configuration/secret/

**Steps to:** Create a Pod that reads and uses a previously 
created Secret object.

* Encode secrets using `base64`:

  - MySQL admin user: `echo -n "admin" | base64` > 
  `YWRtaW4=`
  
  - MySQL admin passwor: 
  `echo -n "mysecretpass" | base64` > `bX1zZWNyZXRwYXNz`

* Create a secret file: `vim mysecret.yaml`:

  ```
  apiVersion: v1
  kind: Secret
  metadata: 
    name: mysecret
  data:
    username: YWRtaW4=
    password: bX1zZWNyZXRwYXNz
  type: Opaque
  ```

* Create the Secret by running: 
  
  ```
  kubectl create -f mysecret.yaml
  ```

* Create a Pod that uses your secret keys by specifying 
`valueFrom: secretKeyRef: name, key`. Pod definition file: 
`vim readsecret.yaml`:

  ```
  apiVersion: v1
  kind: Pod
  metadata:
    name: secret-env-pod
  spec:
    containers:
    - name: mycontainer
      image: redis
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
              optional: false # "mysecret" MUST exist
                              # and include a key named "username"
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password
              optional: false
    
    restartPolicy: Never
  ```

* Create Pod: 
  
  ```
  kubectl create -f readsecret.yaml
  ```

* Check Pod: 
  
  ```
  kubectl get pods
  ```

* You can verify the secrets by logging into the Pod:

  ```
  kubectl exec --stdin --tty secret-env-pod -- /bin/bash
  ```
  
  - Once inside, check that `echo $SECRET_USERNAME` = 
  `admin`

  - Also check that `echo $SECRET_PASSWORD` = 
  `mysecretpass`

---
# Pull an Image from a Private Registry: Docker Hub

Source: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

**Steps to**: Create a Pod that pulls an Image from a 
private registry using a Secret object

* Run `docker login` to log in to your Docker Hub account:
  
  - When prompted, enter your Docker ID, and then the 
  credential you want to use (access token, or the password 
  for your Docker ID)

  - The login process creates or updates a config.json file 
  that holds an authorization token

* `cat ~/.docker/config.json`: View the `config.json` file

  - The output contains a section similar to this:
    
    ```
    {
        "auths": {
            "https://index.docker.io/v1/": {
                "auth": "c3R...zE2"
            }
        }
    }
    ```

* Create a Secret named `regcred.yaml` based on the existing 
credentials saved in `~/.docker/config.json`:
 
  ```
  kubectl create secret generic regcred.yaml \
      --from-file=.dockerconfigjson=path/to/.docker/config.json \
      --type=kubernetes.io/dockerconfigjson
  ```

  - You can set a *Namespace* or a *Label* by modifying the 
  file:
    
      ```
      apiVersion: v1
      kind: Secret
      metadata:
        name: myregistrykey
        namespace: awesomeapps
      data:
        # set the name of the data item to ".dockerconfigjson"
        # base64 encode the Docker configuration file and then paste that string:
        .dockerconfigjson: UmVhbGx5IHJlYWxseSByZWVlZWVlZWVlZWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWFhYWxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGxsbGx5eXl5eXl5eXl5eXl5eXl5eXl5eSBsbGxsbGxsbGxsbGxsbG9vb29vb29vb29vb29vb29vb29vb29vb29vb25ubm5ubm5ubm5ubm5ubm5ubm5ubm5ubmdnZ2dnZ2dnZ2dnZ2dnZ2dnZ2cgYXV0aCBrZXlzCg==
      # set type to kubernetes.io/dockerconfigjson
      type: kubernetes.io/dockerconfigjson
      ```

  - You can base64 encode a file by running: 
  `base64 /path/to/file`

* Create a Pod that uses the Secret by specifying 
`imagePullSecrets`:

  - Here is a manifest for an example Pod that needs access 
  to Docker credentials:

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: private-reg
    spec:
      containers:
      - name: private-reg-container
        image: <your-private-image>
      imagePullSecrets:
      - name: regcred
    ```