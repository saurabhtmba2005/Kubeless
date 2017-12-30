# Kubeless
Use serverless framwork for kubernetes..

Add system:serviceaccount:kubeless:controller-acct account to ABAC policy file ..
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user":"system:serviceaccount:kubeless:controller-acct","nonResourcePath": "*","namespace":"*","resource":"*","apiGroup":"*"}}

Installation

Installation is made of three steps:

    Download the kubeless CLI from the release page. (OSX users can also use brew: brew install kubeless/tap/kubeless).
    Create a kubeless namespace (used by default)
    Then use one of the YAML manifests found in the release page to deploy kubeless. It will create a functions Custom Resource Definition and launch a controller. You will see a kubeless controller, a kafka and a zookeeper statefulset appear and shortly get in running state.

kubectl create ns kubeless
kubectl create -f kubeless.yaml for ABAC kubernetes cluster
kubectl get pods -n kubeless
NAME                                   READY     STATUS    RESTARTS   AGE
kafka-0                                1/1       Running   0          1m
kubeless-controller-3331951411-d60km   1/1       Running   0          1m
zoo-0                                  1/1       Running   0          1m

$ kubectl get deployment -n kubeless
NAME                  DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
kubeless-controller   1         1         1            1           1m

$ kubectl get statefulset -n kubeless
NAME      DESIRED   CURRENT   AGE
kafka     1         1         1m
zoo       1         1         1m

$ kubectl get customresourcedefinition
NAME               KIND
functions.k8s.io   CustomResourceDefinition.v1beta1.apiextensions.k8s.io

$ kubectl get functions
Usage

You can use the CLI to create a function. Functions have three possible types:

    http triggered (function will expose an HTTP endpoint)
    pubsub triggered (function will consume event on a specific topic)
    schedule triggered (function will be called on a cron schedule)

HTTP function

Here is a toy:

def foobar(context):
   print context.json
   return context.json

You create it with:

$ kubeless function deploy get-python --runtime python2.7 \
                                --from-file test.py \
                                --handler test.foobar \
                                --trigger-http
INFO[0000] Deploying function...
INFO[0000] Function get-python submitted for deployment
INFO[0000] Check the deployment status executing 'kubeless function ls get-python'
$ kubeless function ls
NAME           	NAMESPACE	HANDLER              	RUNTIME  	TYPE  	TOPIC      	DEPENDENCIES	STATUS
get-python     	default  	helloget.foo         	python2.7	HTTP  	           	            	1/1 READY

$ kubeless function call get-python --data '{"echo": "echo echo"}'
{"echo": "echo echo"}

Or you can curl directly with kubectl proxy using an apiserver proxy URL. For example:

$ kubectl proxy -p 8080 &

$ curl -L --data '{"Another": "Echo"}' localhost:8080/api/v1/proxy/namespaces/default/services/get-python:function-port/ --header "Content-Type:application/json"
{"Another": "Echo"}

# Function access with Ingress controller 
# Create route
Kubeless support ingress command to create route to function.

$ kubeless route --help
ingress command allows user to list, create, delete routing rule for function on Kubeless

Usage:
  kubeless route SUBCOMMAND [flags]
  kubeless route [command]

Available Commands:
  create      create a route to function
  delete      delete a route from Kubeless
  list        list all routes in Kubeless

Use "kubeless route [command] --help" for more information about a command.

We will create a route to get-python function:

$ kubeless route create route1 --function get-python

$ kubeless route create route2 --function get-python --hostname example.com
$ kubectl get ing
NAME      HOSTS                              ADDRESS          PORTS     AGE
route1    get-python.192.168.99.100.nip.io   192.168.99.100   80        3m
route2    example.com                                         80        6s
You can test the new route with the following command:

$ curl --data '{"Another": "Echo"}' --header "Host: get-python.192.168.99.100.nip.io" 192.168.99.100/ --header "Content-Type:application/json"
{"Another": "Echo"}

# Kubeless ui
$ Kubectl create -f kubelessui.yaml
Url : http://ip:3000

