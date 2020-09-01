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


## Authorization Process
The authorization process is managed by rules, roles, and bindings.

|RBAC Object	| Description |
|-------------|:-----------|
|Rule|	Allowed actions for objects or groups of objects|
|Role	|Sets of rules. Users and groups can be associated with multiple roles.|
|Binding |	Assignment of users or groups to a role.|


RBAC Scope
Red Hat OpenShift Container Platform (RHOCP) defines two groups of roles and bindings depending on the scope and responsibility of users: cluster roles and local roles.

|Role Level	| Description |
|-----------|-------------|
|Cluster Role	| Users or groups with this role level can manage the OpenShift cluster.|
|Local Role	| Users or groups with this role level can only manage elements at a project level.|

## Managing RBAC Using the CLI

- oc adm policy add-cluster-role-to-user __cluster-role__ __username__
- oc adm policy remove-cluster-role-from-user __cluster-role__ __username__

## Default Roles
OpenShift ships with a set of default cluster roles that can be assigned locally or to the entire cluster. You can modify these roles for fine-grained access control to OpenShift resources, but additional steps are required that are outside the scope of this course.

| Default roles	|Description |
|---------------|------------|
|admin	| Users with this role can manage all project resources, including granting access to other users to the project.|
|basic-user |	Users with this role have read access to the project.|
|cluster-admin	| Users with this role have superuser access to the cluster resources. These users can perform any action on the cluster, and have full control of all projects.|
|cluster-status	| Users with this role can get cluster status information.|
| edit	| Users with this role can create, change, and delete common application resources from the project, such as services and deployment configurations. These users cannot act on management resources such as limit ranges and quotas, and cannot manage access permissions to the project.|
|self-provisioner	| Users with this role can create new projects. This is a cluster role, not a project role.|

- oc adm policy add-role-to-user role-name __username__ -n __project__
- oc adm policy add-role-to-user basic-user dev -n wordpress
view	Users with this role can view project resources, but cannot modify project resources.

- oc get clusterrolebinding -o wide
- oc describe clusterrolebindings self-provisioners

```
Name:         self-provisioners
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group  system:authenticated:oauth
  ```

- oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
- oc describe clusterrolebindings self-provisioners
```
Error from server (NotFound): clusterrolebindings.rbac.authorization.k8s.io "self-provisioners" not found
```
- oc policy add-role-to-user admin leader
- oc adm groups new dev-group
- oc adm groups add-users dev-group developer
- oc adm groups new qa-group
- oc adm groups add-users qa-group qa-engineer
- oc get groups
- oc policy add-role-to-group edit dev-group
- oc policy add-role-to-group view qa-group
- oc get rolebindings -o wide
- oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth

## Managing Sensitive Information with Secrets

### Creating a Secret
- oc create secret generic __secret_name__ --from-literal key1=secret1 --from-literal key2=secret2
- Update the pod service account to allow the reference to the secret. to allow a secret to be mounted by a pod running under a specific service account
- oc secrets add --for mount serviceaccount/__serviceaccount-name__ secret/__secret_name__

### Secrets as Pod Environment Variables
```
env:
  - name: MYSQL_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: demo-secret
        key: root_password
```        
- oc set env dc/demo --from=secret/__demo-secret__
### Secrets as Files in a Pod
- oc set volume dc/demo --add --type=secret --secret-name=demo-secret --mount-path=/app-secrets

- oc create secret generic mysql --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql 
- oc new-app --name mysql --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
- oc set env dc/mysql --prefix MYSQL_ --from secret/mysql
```
spec:
      containers:
      - env:
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              key: database
              name: mysql
        - name: MYSQL_HOSTNAME
          valueFrom:
            secretKeyRef:
              key: hostname
              name: mysql
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              key: password
              name: mysql
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              key: user
              name: mysql
```
- oc rsh mysql-2-rqp77
```
sh-4.2$ mysql -u myuser --password=redhat123 test_secrets -e 'show databases;'
sh-4.2$ exit
```
- oc new-app --name quotes --docker-image quay.io/redhattraining/famous-quotes:1.0

- oc get pods -l app=quotes
- oc set env dc/quotes --prefix QUOTES_ --from secret/mysql


## Controlling Application Permissions with Security Context Constraints(SCCs)

Red Hat OpenShift provides security context constraints (SCCs), a security mechanism that restricts access to resources, but not to operations in OpenShift.

SCCs limit the access from a running pod in OpenShift to the host environment. SCCs control:

- Running privileged containers
- Requesting extra capabilities to a container
- Using host directories as volumes
- Changing the SELinux context of a container
- Changing the user ID

```
$ oc get scc
OpenShift provides eight SCCs:

anyuid

hostaccess

hostmount-anyuid

hostnetwork

node-exporter

nonroot

privileged

restricted
$  oc describe scc anyuid
Name:           anyuid
Priority:         10
Access:
  Users:          <none>
  Groups:         system:cluster-admins
Settings:
  Allow Privileged:       false
  Default Add Capabilities:     <none>
  Required Drop Capabilities:     MKNOD,SYS_CHROOT
  Allowed Capabilities:       <none>
  Allowed Volume Types:       configMap,downwardAPI,emptyDir,persistentVolumeClaim,secret
  Allow Host Network:       false
  Allow Host Ports:       false
  Allow Host PID:       false
  Allow Host IPC:       false
  Read Only Root Filesystem:      false
  Run As User Strategy: RunAsAny
    UID:          <none>
    UID Range Min:        <none>
    UID Range Max:        <none>
  SELinux Context Strategy: MustRunAs
    User:         <none>
    Role:         <none>
    Type:         <none>
    Level:          <none>
  FSGroup Strategy: RunAsAny
    Ranges:         <none>
  Supplemental Groups Strategy: RunAsAny
    Ranges:         <none>
```
- oc create serviceaccount __service-account-name__
- oc adm policy add-scc-to-user scc -z __service-account-name__
- oc new-project authorization-scc
- oc new-app --name gitlab --docker-image gitlab/gitlab-ce:8.4-ce.0
- oc get pods
- oc logs pod/gitlab-1-lasdf
```
...output omitted...
================================================================================
Recipe Compile Error in /opt/gitlab/embedded/cookbooks/\
cache/\cookbooks/gitlab/recipes/default.rb
================================================================================

Chef::Exceptions::InsufficientPermissions
-----------------------------------------
directory[/etc/gitlab] (gitlab::default line 26) had an error: \
Chef::Exceptions::InsufficientPermissions: Cannot create directory[/etc/gitlab] at \
/etc/gitlab due to insufficient permissions
...output omitted...
```
- oc create sa gitlab-sa
- oc login -u admin -p XXXXXX
- oc adm policy add-scc-to-user anyuid -z gitlab-sa
- oc login -u developer -p XXX
- oc set serviceaccount deploymentconfig gitlab gitlab-sa
- oc get pods
- oc expose service gitlab --port 80
- oc get route gitlab

## view Roles and Users for a project
```
$ oc get rolebindings
NAME                    ROLE                    USERS     GROUPS                                 SERVICE ACCOUNTS   SUBJECTS
system:image-pullers    /system:image-puller              system:serviceaccounts:asdfasdf4asdf
admin                   /admin                  jsmith
system:deployers        /system:deployer                                                         deployer
system:image-builders   /system:image-builder   
```
## view Roles and Users for the Cluster
```
$ oc get clusterrolebindings
NAME                                            ROLE                                       USERS           GROUPS                                         SERVICE ACCOUNTS                                   SUBJECTS
system:job-controller                           /system:job-controller                                                                                    openshift-infra/job-controller
system:build-controller                         /system:build-controller                                                                                  openshift-infra/build-controller
system:node-admins                              /system:node-admin                         system:master   system:node-admins
registry-registry-role                          /system:registry                                                                                          default/registry
system:pv-provisioner-controller                /system:pv-provisioner-controller                                                                         openshift-infra/pv-provisioner-controller
basic-users                                     /basic-user                                                system:authenticated
system:namespace-controller                     /system:namespace-controller                                                                              openshift-infra/namespace-controller
system:discovery-binding                        /system:discovery                                          system:authenticated, system:unauthenticated
system:build-strategy-custom-binding            /system:build-strategy-custom                              system:authenticated
cluster-status-binding                          /cluster-status                                            system:authenticated, system:unauthenticated
system:webhooks                                 /system:webhook                                            system:authenticated, system:unauthenticated
system:gc-controller                            /system:gc-controller                                                                                     openshift-infra/gc-controller
cluster-readers                                 /cluster-reader                                            system:cluster-readers
system:pv-recycler-controller                   /system:pv-recycler-controller                                                                            openshift-infra/pv-recycler-controller
system:daemonset-controller                     /system:daemonset-controller                                                                              openshift-infra/daemonset-controller
cluster-admins                                  /cluster-admin                             system:admin    system:cluster-admins
system:hpa-controller                           /system:hpa-controller                                                                                    openshift-infra/hpa-controller
system:build-strategy-source-binding            /system:build-strategy-source                              system:authenticated
system:replication-controller                   /system:replication-controller                                                                            openshift-infra/replication-controller
system:sdn-readers                              /system:sdn-reader                                         system:nodes
system:build-strategy-docker-binding            /system:build-strategy-docker                              system:authenticated
system:routers                                  /system:router                                             system:routers
system:oauth-token-deleters                     /system:oauth-token-deleter                                system:authenticated, system:unauthenticated
system:node-proxiers                            /system:node-proxier                                       system:nodes
system:nodes                                    /system:node                                               system:nodes
self-provisioners                               /self-provisioner                                          system:authenticated:oauth
system:service-serving-cert-controller          /system:service-serving-cert-controller                                                                   openshift-infra/service-serving-cert-controller
system:registrys                                /system:registry                                           system:registries
system:pv-binder-controller                     /system:pv-binder-controller                                                                              openshift-infra/pv-binder-controller
system:build-strategy-jenkinspipeline-binding   /system:build-strategy-jenkinspipeline                     system:authenticated
system:deployment-controller                    /system:deployment-controller                                                                             openshift-infra/deployment-controller
system:masters                                  /system:master                                             system:masters
system:service-load-balancer-controller         /system:service-load-balancer-controller    
```


# Chapter 5. Configuring OpenShift Networking Components

## OpenShift Software-defined Networking
- oc describe dns.operator/default
- oc get  Network.config.openshift.io cluster 

## Controlling Cluster Network Ingress

### Describing Methods for Managing Ingress Traffic
- Ingress (resource). The Ingress Operator manages this resource. Ingresses accept external requests and proxy them based on the route. You can only route HTTP, HTTPS and server name identification (SNI), and TLS with SNI.

- External load balancer (service type). This resources instructs OpenShift to spin up a load balancer in a cloud environment. A load balancer instructs OpenShift to interact with the cloud provider in which the cluster is running to provision a load balancer.

- Service external IP (service type). This method instructs OpenShift to set NAT rules to redirect traffic from one of the cluster IPs to the container.

- NodePort (service type). With this method, OpenShift exposes a service on a static port on the node IP address. You must ensure that the external IP addresses are properly routed to the nodes.

### Describing Route Options and Route Types
#### OpenShift Secure Routes
- Edge
- Pass-through
- Re-encryption
-  oc create route edge --service api-frontend --api.apps.acme.com --key api.key --cert api.crt
#### Creating Insecure Routes
-  oc expose service api-frontend

