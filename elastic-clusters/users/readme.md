# Elastic Stack SSO requires a valid Enterprise license so only creating users "manually" is applicable

https://www.elastic.co/guide/en/cloud-on-k8s/master/k8s-users-and-roles.html

# create a folder with the 2 files

```
mkdir filerealm
touch filerealm/users filerealm/users_roles
```

# create user 'myuser' with role 'monitoring_user'
```
docker run \
    -v $(pwd)/filerealm:/usr/share/elasticsearch/config \
    docker.elastic.co/elasticsearch/elasticsearch:8.5.0 \
    bin/elasticsearch-users useradd testuser -p testpasswordmypassword -r superuser
```

# create a Kubernetes secret with the file realm content
`kubectl create secret generic users-secret --from-file filerealm --dry-run -o yaml`

Copy the hashed and base64 encoded values to appropriate values file