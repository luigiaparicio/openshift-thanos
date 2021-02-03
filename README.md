# openshift-thanos

Based on: https://www.openshift.com/blog/federated-prometheus-with-thanos-receive

## Thanos Store Gateway

    oc --context east2 create namespace thanos
    oc --context east2 -n thanos create secret generic store-s3-credentials --from-file=store-s3-secret.yaml
    
    oc --context east2 -n thanos create serviceaccount thanos-store-gateway
    oc --context east2 -n thanos adm policy add-scc-to-user anyuid -z thanos-store-gateway
    
    oc --context east2 -n thanos create -f store-gateway.yaml

    oc --context east2 -n thanos get pods -l "app=thanos-store-gateway"
    
    
## Thanos Receive

    oc --context east2 -n thanos create serviceaccount thanos-receive
    oc --context east2 -n thanos create secret generic thanos-receive-proxy --from-literal=session_secret=$(head /dev/urandom | tr -dc A-Za-z0-9 | head -c43)
    oc --context east2 -n thanos annotate serviceaccount thanos-receive serviceaccounts.openshift.io/oauth-redirectreference.thanos-receive='{"kind":"OAuthRedirectReference","apiVersion":"v1","reference":{"kind":"Route","name":"thanos-receive"}}'
    
    oc --context east2 -n thanos adm policy add-cluster-role-to-user system:auth-delegator -z thanos-receive

    oc --context east2 -n thanos create -f thanos-receive.yaml

    oc --context east2 -n thanos get pods -l "app=thanos-receive"

    oc --context east2 -n thanos create route reencrypt thanos-receive --service=thanos-receive --port=web-proxy --insecure-policy=Redirect

## Create ServiceAccounts for sending metrics

    oc --context east2 -n thanos create serviceaccount west2-metrics
    oc --context east2 -n thanos adm policy add-role-to-user view -z west2-metrics
    oc --context east2 -n thanos create serviceaccount east1-metrics
    oc --context east2 -n thanos adm policy add-role-to-user view -z east1-metrics


    
