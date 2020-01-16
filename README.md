# shinyproxy-kubernetes

Docker image for shinyproxy ment to be used with kubernetes.

Most of the work is just picked from [this GitHub repository.](https://github.com/openanalytics/shinyproxy-config-examples/tree/master/03-containerized-kubernetes) This container is to enable customization in more general way.

## Using a proxy

Proxy env vars are considered and may just be injected using kubernetes env vars. See Example.

## application.yml

*application.yml* is the name of the ShinyProxy configuration file. This file has to be injected using a configmap. AS shinyproxy expects the config to be in the same directory as the *shinyproxy.jar* we do create a symbolic link to the subfolder *./config/application.yml* so the config file can be mounted to this directory.

## Sidecar

Like in the example repo from above, this implementation requires a sidecar to proxy the kubernetes requests.

## Example

Consider:

* Missing all details on *application.yml* config.
* No Ingress/Service configuration which will be required.
* ldap config is omitted.


```yaml
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role 
metadata:
    name: shinyapp-role
    namespace: default
rules:
    - apiGroups: [""]
      resources: ["pods", "pods/log"]
      verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: shinyapp-rolebinding
    namespace: default
roleRef:
    kind: Role 
    name: shinyapp-role
    apiGroup: rbac.authorization.k8s.io 
subjects:
    - kind: ServiceAccount 
      name: default 
      namespace: default

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: shinyapp-config
  namespace: default
  labels:
    app: MyShinyApp
data:
  application.yml: |-
    proxy:
      title: MyShinyApp
      port: 8080
      container-wait-time: 120000
      authentication: ldap
      container-backend: kubernetes
      kubernetes:
        url: 127.0.0.1:8001
        internal-networking: true
        namespace: default
        image-pull-policy: Always
      specs:
      - id: shinyapp
        description: Shiny Application
        xxx: xxx
        container-env:
          http_proxy: "http://your.proxy"
          https_proxy: "http://your.proxy"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: MyShinyApp
  namespace: default
  labels:
    app: MyShinyApp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: MyShinyApp
  template:
    metadata:
      name: MyShinyApp
      labels:
        app: MyShinyApp
    spec:
      automountServiceAccountToken: true
      serviceAccount: default
      containers:
        - name: shinyproxy-sidecar
          image: iptizer/docker-awscli-latest_master
          imagePullPolicy: Always
          command:
            - "kubectl"
            - "proxy"
          ports:
            - containerPort: 8001
        - name: shinyproxy
          image: iptizer/shinyproxy-kubernetes
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
          volumeMounts:
            - name: shinyapp-config-volume
              mountPath: /opt/shinyproxy/config/
      volumes:
        - name: shinyapp-config-volume
          configMap:
            name: shinyapp-config
```
