# ConfigMaps and Secrets

Kubernetes supports configuration injection with two resource types:

- ConfigMaps
- Secrets

Both ConfigMaps and Secrets can store data in most formats. That data lives in the cluster independently of any other resources.

You create ConfigMap and Secret objects with kubectl, either through `create` command or by applying a YAML specification.

Unlike other Kubernetes resources, ConfigMaps and Secrets don't do anything - they're just storage units for small amounts of data. Those storage units can be loaded into a Pod, becoming part of the container environment so the application in the container can read the data.

The simplest way to provide configuration settings is through environment variables.

## Using Environment Variables

In this example, the container includes an environment variable:

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      env:
      - name: KIAMOL_CHAPTER
        value: "04"
```

To print the value of the environment variable, use the following command:

```
kubectl exec deploy/sleep -- printenv HOSTNAME KIAMOL_CHAPTER
```

Setting inline environment variables in the Pod specification is fine for simple cases, but usually you'd have more complex configuration requirements. That's when you need ConfigMaps.

## Using ConfigMaps

A ConfigMap is a resource for storing data that can be loaded into a Pod. The data can be:

- A set of key-value pairs (e.g. environment variables)
- A blurb of text (a config file in JSON, XML, YAML, TOML, INI)
- A binary file (e.g. license keys)

One Pod can use many ConfigMpas and each ConfigMap can be used by many Pods.

The following example uses one environment variable defined in the YAML and a second environment variable loaded from a ConfigMap.

```
env:
- name: KIAMOL_CHAPTER
  value: "04"
- name: KIAMOL_SECTION
  valueFrom:
    configMapKeyRef:
      name: sleep-config-literal # names the ConfigMap
      key: kiamol.section
```

If you reference a ConfigMap in a Pod specification, the ConfigMap needs to exist before you deploy the Pod.

This kubectl command creates a ConfigMap:

```
kubectl create configmap sleep-config-literal --from-literal=kiamol.section='4.1'
```

Check the ConfigMap details:

```
kubectl get cm sleep-config-literal
```

Show a friendly description of the ConfigMap:

```
kubectl describe cm sleep-config-literal
```

Show the Kiamol environment variables:

```
kubectl exec deploy/sleep -- sh -c 'printenv | grep "^KIAMOL"'
```

### Using Configuration Files in ConfigMaps

An environment file is a text file with key-value pairs that can be loade to create one ConfigMap with multiple data items:

```
KIAMOL_CHAPTER=ch04
KIAMOL_SECTION=ch04-4.1
KIAMOL_EXERCISE=try it now
```

The following command loads an environment variable into a new ConfigMap:

```
kubectl create configmap sleep-config-env-file --from-env-file=
sleep/ch04.env
```

:warning: If there are duplicate keys, environment variables define with `env` in the Pod spec overide the values defined with `envFrom`. This means you can override any environment variables set in the container image or in ConfigMaps by setting them explicitly in the Pod spec - useful for debugging! 

### Surfacing Configuration Data from ConfigMaps

An alternative to loading configuration items into environment variables is to present them as files inside directories in the container. 

The container filesystem is a virtual construct, built from the container image and other sources.

Kubernetes can use ConfigMaps as a filesystem source. They are mounted as a directory, with a file for each data item.

This solution is achieved with two features of the Pod spec:

- Volumes - which make the contents of the ConfigMap available to the Pod.
- Volume Mounts - which load the content of the ConfigMap volume into a specified path in the Pod container.

Here's an example:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-web
spec:
  selector:
    matchLabels:
      app: todo-web
  template:
    metadata:
      labels:
        app: todo-web
    spec:
      containers:
        - name: web
          image: kiamol/ch04-todo-list 
          volumeMounts:
            - name: config
              mountPath: "/app/config" # directory path to mount the volume
              readOnly: true
      volumes:
        - name: config # name matches the volumeMount
          configMap:
            name: todo-web-config-dev
```

In the example above:

- The container filesystem is constructed by Kubernetes.
- The /app directory is loaded from the container image.
- The /app/config directory is loaded from the ConfigMap.

The ConfigMap is treated like a directory. It's multipe data items each become files in the container filesystem.

:warning: If the mount path for a volume already exists in the container image, then the ConfigMap directory overwrites it and replaces all the contents, you app might fail in exciting ways.

You can selectively load data items into the target directory, rather than loading every data item as its own file.

For example:

```
 spec:
    containers:
      - name: web
        image: kiamol/ch04-todo-list
        volumeMounts:
          - name: config
            mountPath: "/app/config"
            readOnly: true
    volumes:
      - name: config
        configMap:
          name: todo-web-config-dev
          items:
          - key: config.json
            path: config.json
```

ConfigMaps are effectively wrappers for text files, so you shouldn't use them for sensitive data.

## Using Secrets

Secrets work similarly to ConfigMaps, but Kubernetes manages them differently - they're designed to minimise exposure:

-  Secrets are sent only to nodes that need to use them.
-  Secrets are stored in memory, rather than on disk. 
-  Secrets can be encrypte both in transit and at rest.

The following command creates a Secret:

```
kubectl create secret generic sleep-secret-literal --from-literal=secret=shh...
```

Show the friendly details of the Secret:

```
kubectl describe secret sleep-secret-literal
```

Retrieve the encoded Secret value:

```
kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}'
```

Decode the data:

```
kubectl get secret sleep-secret-literal -o jsonpath='{.data.secret}' | base64 -d
```

Describing the Secret shows only the key and the data length, not the value. Kubernetes tries to avoid accidental exposure. This doesn't apply when Secrets are surfaced within Pod container - the container environment sees the original plain text data.

The following example shows an app configured to load a Secret as an environment variable:

```
spec:
  containers:
    - name: sleep
      image: kiamol/ch03-sleep
      env:
      - name: KIAMOL_SECRET
        valueFrom:
          secretKeyRef:              
            name: sleep-secret-literal
            key: secret
```

The following command prints the name of the environment variable:

```
kubectl exec deploy/sleep -- printenv KIAMOL_SECRET
```

Secrets and ConfigMaps can be mixed in the same Pod spec, populating environment variables or files, or both.

:warning: You should avoid loading Secrets into environment variables. Environment variables can be read from any process in the Pod container and some application platforms log all environment variable values if they encounter a critical error. The alternative is to surface Secrets as files, if the application supports it, which gives you the option to secure access with file permissions.

