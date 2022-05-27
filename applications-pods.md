# Working with Applications in Pods

Although Deployments are managing your app, the real work happens in the container. You can work with containers in Pods using kubectl. Through the command line, you can:

- Run commands in containers.
- View application logs.
- Copy files

The following command prints the Pod's IP address:

`kubectl get pod hello-kiamol -o custom-columns=NAME:metadata.name,POD_IP:status.podIP`

## Running an Interactive Shell in the Pod

With this command, you can run an interactive shell in the Pod:

`kubectl exec -it hello-kiamol -- sh`

Check the IP address:

`kubectl exec -it hello-kiamol -- sh`

and see that it matches the previous results.

This command tests the web app:

`wget -O - http://localhost | head -n 4`

Type `exit` to leave the shell.

Running an interactive shell inside a Pod container is useful for understanding what's happening from that Pod's perspective. You can:

- Read file contents to check that configuration settings are applied correctly.
- Run DNS queries to verify that services are resolving as expected.
- Ping endpoints to test the network.

## Reading the Application Logs

For ongoing administration, it's simpler to read the application logs, rather than running the interactive shell.

Kubernetes fetches application logs from the container runtime. You can read logs with kubectl and, if you have access to the container runtime, you can verify that they're the same as the container logs.

The following command prints the latest container logs:

`kubectl logs --tail=2 hello-kiamol`

The `tail` parameter restricts the output to the two most recent log entries.

If you're using Docker, this command compare the container log with the container runtime:

`docker container logs --tail=2 $(docker container ls -q --filter label=io.kubernetes.container.name=hello-kiamol)`

Pods managed by controllers have random names, so you don't refer to them directly. Instead, you can access them by their controller or by their labels.

The following command makes a call to the web app inside a container. The Pod was created from a Deployment:

`kubectl exec deploy/hello-kiamol-4 -- sh -c 'wget -O - http://localhost > /dev/null'`

This command prints the most recent log entry for that container:

`kubectl logs --tail=1 -l app=hello-kiamol-4`

The entry refers to the previous call.

## Copying Files from Containers

kubectl lets you copy files between your local machine and containers in Pods. 

The following command copies the web page from the Pod's container:

`kubectl cp hello-kiamol:/usr/share/nginx/html/index.html /tmp/kiamol/ch02/index.html`

You can reverse this command to copy a file from your local machine to the Pod's container.

### Key Points

- You can work with Pods using kubectl without knowing the Pod's name - you can access them by their controller or labels.