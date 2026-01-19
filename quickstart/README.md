kubectl create namespace argo

kubectl apply -n argo -f "https://github.com/argoproj/argo-workflows/releases/download/${ARGO_WORKFLOWS_VERSION}/quick-start-minimal.yaml"

ARGO_WORKFLOWS_VERSION="vX.Y.Z" # v3.6.15

kubectl apply -n argo -f "https://github.com/argoproj/argo-workflows/releases/download/v3.6.15/quick-start-minimal.yaml"


argo submit -n argo --watch https://raw.githubusercontent.com/argoproj/argo-workflows/main/examples/hello-world.yaml

argo list -n argo

argo get -n argo @latest

argo logs -n argo @latest

kubectl -n argo port-forward service/argo-server 2746:2746