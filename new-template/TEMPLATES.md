There are several types of templates, divided into two different categories: work and orchestration.

The first category defines work to be done. This includes:

Container
Container Set
Data
Resource
Script
The second category orchestrates the work:

DAG
Steps
Suspend
Next, we're going to take a deep-dive into the two most common template types: Container and DAG.


Container Template
A container template is the most common type of template, let's look at a complete example:

apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: container-
spec:
  entrypoint: main
  templates:
  - name: main
    container:
      image: busybox
      command: [echo]
      args: ["hello world"]
Let's run the workflow:

argo submit --serviceaccount argo-workflow --watch container-workflow.yaml

Port-forward to the Argo Server pod...

kubectl -n argo port-forward --address 0.0.0.0 svc/argo-server 2746:2746 > /dev/null &

and open the Argo Workflows UI. Then navigate to the workflow, you should see a single container running.

Exercise
Edit the workflow to make it echo "howdy world".


Template Tags
Template tags (also known as template variables) are a way for you to substitute data into your workflow at runtime. Template tags are delimited by {{ and }} and will be replaced when the template is executed.

What tags are available to use depends on the template type, and there are a number of global ones you can use, such as {{workflow.name}}, which is replaced by the workflow's name:

    - name: main
      container:
        image: busybox
        command: [ echo ]
        args: [ "hello {{workflow.name}}" ]
Look at the full example:

cat template-tag-workflow.yaml

Submit this workflow:

argo submit --serviceaccount argo-workflow --watch template-tag-workflow.yaml

You can see the output by running

argo logs @latest

You should see something like:

hello template-tag-kqpc6
There are many more different tags, you can read more about template tags in the docs.

Exercise
Change the workflow to echo the date the workflow was created.


Work Templates
What other types of work templates are there?

A container set allows you to run multiple containers in a single pod. This is useful when you want the containers to share a common workspace, or when you want to consolidate pod spin-up time into one step in your workflow.

A data template allows you get data from storage (e.g. S3). This is useful when each item of data represents an item of work that needs doing.

A resource template allows you to create a Kubernetes resource and wait for it to meet a condition (e.g. successful) . This is useful if you want to interoperate with another Kubernetes system, like AWS Spark EMR.

A script template allows you to run a script in a container. This is very similar to a container template, but when you've added a script to it.

Every type of template that does work, does so by running a pod. You can use kubectl to view these pods:

kubectl get pods -l workflows.argoproj.io/workflow

You can identify workflow pods by the workflows.argoproj.io/workflow label.

You should see something like this:

NAME                 READY   STATUS      RESTARTS   AGE
container-m5664      0/2     Completed   0          5m21s
template-tag-kqpc6   0/2     Completed   0          4m6s
Exercise
Use kubectl describe to describe a workflow pod. What interesting information is contained within the pod's labels and annotations?


DAG Template
A DAG template is a common type of orchestration template. Let's look at a complete example:

apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: dag-
spec:
  entrypoint: main
  templates:
    - name: main
      dag:
        tasks:
          - name: a
            template: echo
          - name: b
            template: echo
            dependencies:
              - a
    - name: echo
      container:
        image: busybox
        command: [ echo ]
        args: [ "hello world" ]

In this example, we have two templates:

The "main" template is our new DAG.
The "echo" template is the same template as in the container example.
The DAG has two tasks: "a" and "b". Both run the "echo" template, but as "b" depends on "a", it won't start until " a" has completed successfully.

Let's run the workflow:

argo submit --serviceaccount argo-workflow --watch dag-workflow.yaml

You should see something like:

STEP          TEMPLATE  PODNAME              DURATION  MESSAGE
 ✔ dag-shxn5  main
 ├─✔ a        echo       dag-shxn5-289972251  6s
 └─✔ b        echo       dag-shxn5-306749870  6s
Did you see how b did not start until a had completed?

Open the Argo Server tab and navigate to the workflow, you should see two containers.

Exercise
Add a new task named "c" to the DAG. Make it depend on both "a" and "b". Go to the UI and view your updated workflow graph.


Loops
The ability to run large parallel processing jobs is one of the key features of Argo Workflows. Let's have a look at using loops to do this.

withItems
A DAG allows you to loop over a number of items using withItems :

      dag:
        tasks:
          - name: print-message
            template: echo
            arguments:
              parameters:
                - name: message
                  value: "{{item}}"
            withItems:
              - "hello world"
              - "goodbye world"
In this example, it will execute once for each of the listed items. We can see a template tag here. {{item}} will be replaced with "hello world" and "goodbye world". DAGs execute in parallel, so both tasks will be started at the same time.

argo submit --serviceaccount argo-workflow --watch with-items-workflow.yaml

You should see something like:

STEP                                 TEMPLATE  PODNAME                      DURATION  MESSAGE
 ✔ with-items-4qzg9                  main
 ├─✔ print-message(0:hello world)    echo  with-items-4qzg9-465751898   7s
 └─✔ print-message(1:goodbye world)  echo  with-items-4qzg9-2410280706  5s
Notice how the two items ran at the same time.

withSequence
You can also loop over a sequence of numbers using withSequence :

      dag:
        tasks:
          - name: print-message
            template: echo
            arguments:
              parameters:
                - name: message
                  value: "{{item}}"
            withSequence:
              count: 5
As usual, run it:

argo submit --serviceaccount argo-workflow --watch with-sequence-workflow.yaml

STEP                     TEMPLATE  PODNAME                         DURATION  MESSAGE
 ✔ with-sequence-8nrp5   main
 ├─✔ print-message(0:0)  echo  with-sequence-8nrp5-3678575801  9s
 ├─✔ print-message(1:1)  echo  with-sequence-8nrp5-1828425621  7s
 ├─✔ print-message(2:2)  echo  with-sequence-8nrp5-1644772305  13s
 ├─✔ print-message(3:3)  echo  with-sequence-8nrp5-3766794981  15s
 └─✔ print-message(4:4)  echo  with-sequence-8nrp5-361941985   11s
See how 5 pods were run at the same time, and that their names have the item value in them, zero-indexed?

Exercise
Change the withSequence to print the numbers 10 to 20.


Orchestration Templates
We learned that a DAG template is a type of orchestration template. What other types of orchestration templates are there?

A steps template allows you to run a series of steps in sequence.

A suspend template allows you to automatically suspend a workflow, e.g. while waiting on manual approval, or while an external system does some work.

Orchestration templates do NOT run pods. You can check by running kubectl get pods .


Exit Handler
If you need to perform a task after something has finished, you can use an exit handler. Exit handlers are specified using onExit :

      dag:
        tasks:
          - name: a
            template: echo
            onExit: tidy-up
They just state the name of the template that should be run on-exit.

Let's look at a complete example:

cat exit-handler-workflow.yaml

Run it:

argo submit --serviceaccount argo-workflow --watch exit-handler-workflow.yaml

You should see:

STEP                   TEMPLATE  PODNAME                        DURATION  MESSAGE
 ✔ exit-handler-plvg7  main
 ├─✔ a                 echo      exit-handler-plvg7-1651124468  5s
 └─✔ a.onExit          tidy-up   exit-handler-plvg7-3635807335  6s
Note how the exit handler task ran last.

Exercise
An exit handler can be run at the end of a template, or at the end of a workflow. Change the example to run one at the end of the workflow.

Learn more about exit handlers, as well as their close cousin, lifecycle hooks, in the Argo Workflows documentation.