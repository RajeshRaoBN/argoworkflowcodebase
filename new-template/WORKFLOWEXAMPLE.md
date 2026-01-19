Workflow Examples
This is a free-form scenario that collects all the Argo Workflows examples for you and allows you to run them.

Please allow Argo Workflows time to install before continuing.


Argo Workflows Examples
The examples are pulled directly from the official argoproj/argo-workflows repository. This means it is possible for an example to be broken. If you find an example that doesn't work, and you have tested outside of Killercoda, please open an issue on the official repository.

If an example works outside of Killercoda, but not in the course, please open an issue on the course repository.

List all examples
ls /root/examples

Run an example
argo submit -n argo --serviceaccount argo-workflow --watch /root/examples/coinflip.yaml

You may need to kubectl apply additional files to the cluster prior to running an example
kubectl apply -n argo -f /root/examples/workflow-template/templates.yaml

View the server UI
kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &

click here to access the UI


