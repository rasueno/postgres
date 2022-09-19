# Create postgres db instance

1. Create namespace
    `kubectl create namespace postgres`
2. Create Persistent Volume
    `kubectl apply -n postgres -f postgres-pv.yml`
3. Create Persistent Volume Claim
    `kubectl apply -n postgres -f postgres-pvc.yml`
4. Install bitnami postgress
    `helm install postgres-dev bitnami/postgresql -n postgres --set primary.persistence.existingClaim=postgresql-pv-claim --set volumePermissions.enabled=true`
5. Update istio system to expose postgres in a new IP
    `istioctl manifest generate -f istio-operator.yml | kubectl apply -n istio-system -f -`
6. Apply new gateway and virtualservice
    `kubectl apply -n postgres -f postgres-gw.yml`
    `kubectl apply -n postgres -f postgres-vs.yml`

## Get the password
`export POSTGRES_PASSWORD=$(kubectl get secret --namespace postgres postgres-dev-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)`

## Connect to the database
`kubectl run postgres-dev-postgresql-client --rm --tty -i --restart='Never' --namespace postgres --image docker.io/bitnami/postgresql:14.5.0-debian-11-r14 --env="PGPASSWORD=$POSTGRES_PASSWORD" --command -- psql --host postgres-dev-postgresql -U postgres -d postgres -p 5432`

## Connect from outside the cluster
`kubectl port-forward --namespace postgres svc/postgres-dev-postgresql 5432:5432 & PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432`