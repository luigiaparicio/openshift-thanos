# openshift-thanos


    oc --context east2 create namespace thanos
    oc --context east2 -n thanos create secret generic store-s3-credentials --from-file=store-s3-secret.yaml
