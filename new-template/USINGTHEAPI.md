Using the API
In this scenario you will:

Learn how to create an access token to connect to the API.
Learn how to submit a workflow using curl.
Learn how to look up API documentation.


Info Endpoint
The Argo Server provides the API. This is secured using Kubernetes service accounts.

All endpoints can be found under http://localhost:2746/api/v1 URL and typically require an access token.

To access this via Killercoda, we need to port-forward the argo-server pod:

kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &

Note: Killercoda currently has issues port-forwarding. You may need to wait a few minutes for Killercoda to complete the port-forward.

Typically, it is good to be able to check you can access the API before using it. This can be done using the info endpoint:

curl http://localhost:2746/api/v1/info

You should see something like this if it is successful:

{"code":16,"message":"token not valid. see https://argo-workflows.readthedocs.io/en/latest/faq/"}
If it fails, then you'll see something like this:

curl: (7) Failed to connect to localhost port 2746: Connection refused
Note: If see a failure message, it is likely a Killercoda issue. You should try re-submitting the kubectl -n argo port-forward command listed above, and then repeat the steps to confirm your port-forward was successful before moving to the next section.

To connect, we need to set-up an access token.


Access Token
If you want to automate tasks with the Argo Server API or CLI, you will need an access token. An access token is just a Kubernetes service account token. So, to set up a service account for our automation, we need to create:

A role with the permission we want to use.
A service account for our automation user.
A service account token for our service account.
A role binding to bind the role to the service account.
In our example, we want to create a role for Jenkins so it can create, get and list workflows:

Create the role:

kubectl create role jenkins --verb=create,get,list --resource=workflows.argoproj.io --resource=workfloweventbindings --resource=workflowtemplates

Create the service account:

kubectl create sa jenkins

Bind the service account to the role:

kubectl create rolebinding jenkins --role=jenkins --serviceaccount=argo:jenkins

Now we can create a token:

ARGO_TOKEN="Bearer $(kubectl create token jenkins)"

Print out the token:

echo $ARGO_TOKEN

You should see something like the following:

Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6...
To use the token, you add it as an Authorization header to your HTTP request:

curl http://localhost:2746/api/v1/info -H "Authorization: $ARGO_TOKEN"

You should see something like the following:

{"modals":{"feedback":false,"firstTimeUser":false,"newVersion":false}}...
Now you are ready to create an Argo Workflow using the API.


Create a Workflow
API endpoints take JSON rather than YAML as their payload, you can submit a workflow using curl :

curl \
   http://localhost:2746/api/v1/workflows/argo \
  -H "Authorization: $ARGO_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
  "workflow": {
    "metadata": {
      "generateName": "hello-world-"
    },
    "spec": {
      "templates": [
        {
          "name": "main",
          "container": {
            "image": "busybox",
            "command": [
              "echo"
            ],
            "args": [
              "hello world"
            ]
          }
        }
      ],
      "entrypoint": "main"
    }
  }
}'
You should see something like:

{"metadata":{"name":"hello-world-mmbfs",...
We can wait for that workflow to complete by polling:

curl \
    http://localhost:2746/api/v1/workflows/argo/@latest \
    -H "Authorization: $ARGO_TOKEN"
You should see something like:

{"metadata":{"name":"hello-world-2j74f",...
Exercise
Create a workflow template and then submit the workflow template using the API.


Webhooks
To keep things simple, we used the api/v1/workflows endpoint to create workflows, but there's one endpoint that is specifically designed to create workflows via an api: api/v1/events . You should use this for most cases (including Jenkins):

It only allows you to create workflows from a WorkflowTemplate , so is more secure.
It allows you to parse the HTTP payload and use it as parameters.
It allows you to integrate with other systems without you having to change those systems.
Webhooks also support GitHub and GitLab, so you can trigger workflow from git actions.
To use this, you need to create a WorkflowTemplate and a workflow event binding.:

A workflow event binding consists of:

An event selector that matches events
A reference to a WorkflowTemplate using workflowTemplateRef
Optional parameters
Example:

apiVersion: argoproj.io/v1alpha1
kind: WorkflowEventBinding
metadata:
  name: hello
spec:
  event:
    selector: payload.message != ""
  submit:
    workflowTemplateRef:
      name: hello
    arguments:
      parameters:
        - name: message
          valueFrom:
            event: payload.message
In the above example, if the event contained a message, then we'll submit the workflow template and the workflow will echo the message.

Create the WorkflowTemplates :

kubectl apply -f hello-workflowtemplate.yaml

Create the workflow event binding:

kubectl apply -f hello-workfloweventbinding.yaml

Try it out:

curl http://localhost:2746/api/v1/events/argo/- -H "Authorization: $ARGO_TOKEN" -d '{"message": "hello events"}'

You will not get a response - this is processed asynchronously.

Allow about 5 seconds for the workflow to start and then check the logs:

argo logs @latest

You should see something like the following:

hello-u7mnk: hello events
Learn more about webhooks in the Argo Workflows docs.

Exercise
Update the workflow event binding to add annotations to created workflows. Hint: read the docs linked above.


API Docs
You can find API docs in the user interface, try it now:

Open the API docs

You can use the API docs to send API requests, so it is really useful to test things out.

But you were asked for a password weren't you?

Use your ARGO_TOKEN as a password:

echo $ARGO_TOKEN

Copy the whole response, including Bearer , and paste it into the white box in the center of the Argo UI Login page where it says "client authentication".

Once you're logged in, you may need to click to open the API docs again:

Open the API docs

Exercise
Open the API docs and find an endpoint to create workflow templates. Create a workflow template and then submit it.

More API documentation
Visit the Argo Workflows documentation for more information:

Argo Workflows API
Access Tokens
API examples
API reference (Swagger docs)
Client libraries for Python, Golang, and Java
Argo Server


Let's recap:

You can use the info endpoint to check connectivity.
You need to create an access token to use the API.
API endpoints use JSON not YAML.
Webhooks allow you integrate with any service that supports them.
You should use the api/v1/events endpoint for create workflows.
You can find API docs in the user interface.