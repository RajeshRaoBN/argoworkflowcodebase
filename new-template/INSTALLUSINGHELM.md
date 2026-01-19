Install using Helm
Installing Argo Workflows with Helm
This command installs the latest Argo Workflows Helm Chart. In production it would be sensible to pin the version of the chart to a specific version. Argo is normally installed into a namespace named argo so we will create that as part of our deployment:

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argo argo/argo-workflows \
  --namespace argo \
  --create-namespace \
  --set workflow.serviceAccount.create=true
What was installed?
It will take about 1m for all deployments to become available. Let's look at what is installed while we wait.

The Workflow Controller is responsible for running workflows:

kubectl -n argo get deploy argo-argo-workflows-workflow-controller

And the Argo Server provides a user interface and API:

kubectl -n argo get deploy argo-argo-workflows-server

Wait for everything to be ready
Before we proceed, let's wait (around 1 minute to 2 minutes) for our deployments to be available:

kubectl -n argo wait deploy --all --for condition=Available --timeout 2m


Uninstalling Argo Workflows
If we wish to uninstall Argo Workflows, we can do so using Helm:

helm uninstall argo --namespace argo


Modifying the install
Helm allows us to configure our deployment by modifying the default values.

The argo-server (and thus the UI) defaults to client authentication, which requires clients to provide their Kubernetes bearer token in order to authenticate. For more information, refer to the Argo Server Auth Mode documentation.

We will switch the authentication mode to server so that we can bypass the UI login for now. This is not something we recommend for production installs.

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argo argo/argo-workflows --namespace argo --create-namespace \
  --set server.extraArgs[0]="--auth-mode=server" \
  --set workflow.serviceAccount.create=true
We need to wait for the Argo Server to deploy:

kubectl -n argo rollout status --watch --timeout=600s deployment/argo-argo-workflows-server

You can then view the user interface by running a port forward:

kubectl -n argo port-forward --address 0.0.0.0 svc/argo-argo-workflows-server 2746:2746 > /dev/null &

You can then click here to access the UI. As it's your first time using the Workflows UI, you will see a number of modals explaining the new features. Dismiss them.

Run a workflow
Open the "Argo Server" tab and you should see the user interface:

ui.png

Lets start a workflow from the user interface:

Click "Submit new workflow":

submit-01.png

Click "Edit using full workflow options". You should see something similar to this:

submit-02.png

Paste this YAML into the editor:

metadata:
  generateName: hello-world-
  namespace: argo
spec:
  serviceAccountName: argo-workflow
  entrypoint: main
  templates:
    - name: main
      container:
        image: busybox
        command: ["echo"]
        args: ["hello world"]
Click "Create". You will see a diagram of the workflow. The yellow icon shows that it is pending, after a few seconds it'll turn blue to indicate it is running, and finally green to show that it has completed successfully:

running.png

After about 30s, the icon will change to green:

green.png


Let's recap:

Argo Workflows can be installed using Helm.
It can be uninstalled using Helm.
The installation can be customized by modifying the Helm values.