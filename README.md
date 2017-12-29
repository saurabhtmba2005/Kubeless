# Kubeless
Use serverless framwork for kubernetes..

Add system:serviceaccount:kubeless:controller-acct account to ABAC policy file ..
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user":"system:serviceaccount:kubeless:controller-acct","nonResourcePath": "*","namespace":"*","resource":"*","apiGroup":"*"}}

Installation

Installation is made of three steps:

    Download the kubeless CLI from the release page. (OSX users can also use brew: brew install kubeless/tap/kubeless).
    Create a kubeless namespace (used by default)
    Then use one of the YAML manifests found in the release page to deploy kubeless. It will create a functions Custom Resource Definition and launch a controller. You will see a kubeless controller, a kafka and a zookeeper statefulset appear and shortly get in running state.
# Kubeless ui

