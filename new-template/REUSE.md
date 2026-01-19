Reuse
In this scenario you will:

Learn about reuse using workflow templates.
Learn about running a workflow on schedule using cron workflows.


Workflow Templates
Workflow templates (not to be confused with a template) allow you to create a library of code that can be reused. They're similar to pipelines in Jenkins.

Workflow templates have a different kind to a workflow, but are otherwise very similar:

apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: hello
spec:
  entrypoint: main
  templates:
    - name: main
      container:
        image: busybox
        command: [ echo ]
        args: [ "hello world" ]
Let's create this workflow template:

argo template create hello-workflowtemplate.yaml

You could also manage templates using kubectl :

kubectl create -f hello-workflowtemplate.yaml

This allows you to use GitOps to manage your templates.

To submit (i.e. run) a workflowTemplate, you can use the UI or the CLI:

argo submit --watch --from workflowtemplate/hello

You should see:

STEP            TEMPLATE  PODNAME      DURATION  MESSAGE
 ✔ hello-c622t  main      hello-c622t  33s
Lets take a look at the workflow you created:

argo get @latest -o yaml

Look for the workflow specification in the output:

spec:
  workflowTemplateRef:
    name: hello
Note how the specification of the workflow is actually a reference to the template.

Exercise
Use the user interface to submit a workflow template:
Port-forward to the Argo Server pod... kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &
and open the Argo Workflows UI.
Update the workflow template to add some parameters (e.g. to print a message). Use argo submit --from to submit it with different parameters.


Cron Workflows
A cron workflow is a workflow that runs on a cron schedule:

apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: hello-cron
spec:
  schedule: "* * * * *"
  workflowSpec:
    entrypoint: main
    templates:
      - name: main
        container:
          image: busybox
          command: ["echo"]
          args: ["hello world"]
When it should be run is set in the schedule field, in the example every minute.

Lets created this cron workflow:

argo cron create hello-cronworkflow.yaml

You'll need to wait for up to a minute to see the workflow run.

Exercise
Cron workflows can be submitted immediately from the CLI or the UI. Find out how.


Let's recap:

A workflow template is a workflow that can be reused or as a library item.
A cron workflow is a workflow that runs on a schedule.