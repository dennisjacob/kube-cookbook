```
NAMESPACE=istio-ns
kubectl get ns $NAMESPACE -o json |  jq '.spec.finalizers=[]' |  curl -X PUT http://localhost:8001/api/v1/namespaces/${NAMESPACE}/finalize -H "Content-Type: application/json" --data @-
```
