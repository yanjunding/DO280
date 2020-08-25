# Troubleshooting commands

- oc get nodes
- oc adm top node -l node-role.kubernetes.io/worker
- oc describe node ip-10-0-20-1-ap-southeast-1.computer.internel
- oc get pod -n openshift-image-registry
- oc logs --tail 3 -n openshift-iamge-registry cluster-image-registry-operator-xxxx-xxxx
- oc logs --tail 3 -n openshift-iamge-rgeistry -c cluster-image-registry-operator cluster-image-rgeistry-operator-xxxx-xxxx
- oc logs --tail 3 -n openshift-image-registry iamge-registry-xxx-yyyy
- oc adm node-logs --tail 3 -u kebelet ip-1.2.2..computer.internel
- oc debug node/ip-1.2.2..computer.internel
- oc status
- oc get events

# 3. Configuring Authentication

## Users

### Regular users

This is the way most interactive OpenShift Container Platform users are represented. Regular users are created automatically in the system upon first login or can be created via the API. Regular users are represented with the User object. Examples: joe alice

### System users

Many of these are created automatically when the infrastructure is defined, mainly for the purpose of enabling the infrastructure to interact with the API securely. They include a cluster administrator (with access to everything), a per-node user, users for use by routers and registries, and various others. Finally, there is an anonymous system user that is used by default for unauthenticated requests. Examples: system:admin system:openshift-registry system:node:node1.example.com

### Service accounts

These are special system users associated with projects; some are created automatically when the project is first created, while project administrators can create more for the purpose of defining access to the contents of each project. Service accounts are represented with the ServiceAccount object. Examples: system:serviceaccount:default:deployer system:serviceaccount:foo:builder

## Groups

### system:authenticated

Automatically associated with all authenticated users.

### system:authenticated:oauth

Automatically associated with all users authenticated with an OAuth access token.

### system:unauthenticated

Automatically associated with all unauthenticated users.


## commands


- htpasswd -C -B -b ~/temp admin USER_PASSWD
- htpasswd -b ~/temp developer USER_PASSWD
- cat ~temp
- oc login -u kuberadmin -p KUBEADM_PASSWD MASTER_API
- oc create secret generic localusers --from-file htpasswd=~/temp -n openshift-config
- oc adm policy add-cluster-role-to-user cluster-admin admin
- oc get -o yaml oauth cluster > ~/oauth.yaml

```
apiVersion: config.openshift.io/v1
kind: OAuth
...output omitted...
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd
```
- oc replace -f ~/oauth.yaml

- oc login -u admin -p USER_PASSWD 
- oc get nodes
- oc login -u developer -p USER_PASSWD
- oc get nodes

```Error from server (Forbidden): nodes is forbidden: User "developer" cannot list
resource "nodes" in API group "" at the cluster scope
```
- oc login -u admin -p USER_P
- oc get users
- oc get identity

```
NAME                IDP NAME   IDP USER NAME   USER NAME
myusers:admin       myusers    admin           admin       ...
myusers:developer   myusers    developer       developer   ...
```
- oc extract secret/localusers -n openshift-config --to -
- htpasswd -b ~/temp manager USER_PASSWD
- cat ~/temp

```
admin:$2y$05$QPuzHdl06IDkJssT.tdkZuSmgjUHV1XeYU4FjxhQrFqKL7hs2ZUl6
developer:$apr1$0Nzmc1rh$yGtne1k.JX6L5s5wNa2ye.
manager:$apr1$CJ/tpa6a$sLhjPkIIAy755ZArTT5EH/
```
- oc login -u manager -p USER_PASSWD
- oc new-project auth-provider
- oc login -u developer -p USER_PASSWD
- oc delete project auth-provider

```
Error from server (Forbidden): projects.project.openshift.io "auth-provider"
is forbidden: User "developer" cannot delete resource "projects"
in API group "project.openshift.io" in the namespace "auth-provider"
```
- oc lgoin -u admin -p USER_PASSWD
- oc extract secret/localusers -n openshift-config --to -

- oc edit oauth
- oc delete secret localusers -n openshift-config

- oc delete user --all
- oc delete identity --all

# Controlling Access to Openshift Resources



