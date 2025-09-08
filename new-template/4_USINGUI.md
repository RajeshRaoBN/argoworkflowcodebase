Using the UI
The argo-server (and thus the UI) defaults to client authentication, which requires clients to provide their Kubernetes bearer token in order to authenticate. For more information, refer to the Argo Server Auth Mode documentation.

We will switch the authentication mode to server so that we can bypass the UI login for now.

Additionally, Argo Server runs over https by default. This isn't compatible with Killercoda, so we will disable https at the same time. This is not something we recommend for production installs.

kubectl patch deployment \
  argo-server \
  --namespace argo \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/args", "value": [
  "server",
  "--auth-mode=server",
  "--secure=false"
]},
{"op": "replace", "path": "/spec/template/spec/containers/0/readinessProbe/httpGet/scheme", "value": "HTTP"}
]'
We need to wait for the Argo Server to redeploy:

kubectl -n argo rollout status --watch --timeout=600s deployment/argo-server

You can then view the user interface by running a port forward:

kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &

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
Click "Create". You will see a diagram of the workflow. The yellow icon shows that it is pending, after a few seconds it'll turn blue to indicate it is running, and finally green to show that it has completed successfully:

running.png

After about 30s, the icon will change to green:

green.png

Exercise
Take a few minutes to play around with the user interface. Find out how to:

List workflows.
View a workflow.
Resubmit a completed workflow.