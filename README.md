## OpenLDAP

Installs OpenLDAP and phpLDAPadmin with a small number of initial users for the purposes of demonstrating LDAP integration capabilities of ICP.

## Updated Installation 

This update allows the chart can to be used with latest version of OCP/Kubernetes
with updates to the API version of deployments. It also removes the use of `hostPath` and uses emptyDir instead so no host scc's are needed. Tested with Helm 3. Refer to `Setup IBM Common Services LDAP integration` section to add LDAP to common services.

```
# Set Variables
LDAP_PROJECT=cp4i-ldap
RELEASE=cs-ldap

# New project and add SCC
oc new-project $LDAP_PROJECT
oc adm policy add-scc-to-user anyuid -z default -n $LDAP_PROJECT

# Install Helm Chart
helm install $RELEASE .

# Check if pods are up, and if so, port-forward to test the LDAP connection
oc port-forward svc/$RELEASE 1389:389

# Test LDAP
ldapsearch -x -H ldap://localhost:1389 -b 'dc=local,dc=io' -D "cn=admin,dc=local,dc=io" -w admin

# Test Admin UI
oc port-forward svc/$RELEASE-admin 1880:80

# Login to localhost:1880 from your browser, and login with default creds (unless modified)
# username: cn=admin,dc=local,dc=io
# password: admin
```

To Add to Common Service, the LDAP server URL is `ldap://cs-ldap.cp4i-ldap.svc:389`. 

## Installation (Previous Instructions)
To install the chart, you'll need the [helm cli](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/app_center/create_helm_cli.html?view=kc) and the [IBM Cloud Private CLI](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/manage_cluster/install_cli.html?view=kc). Note: the IBM Cloud Private CLI version level must match the version level that is downloadable via your ICP console, under ***Menu > Command Line Tools > Cloud Private CLI***.

1. Get the source code of the helm chart
   `git clone https://github.com/ibm-cloud-architecture/icp-openldap.git`
2. Package the helm chart using the helm cli
   `helm package icp-openldap`
3. If you have not, log in to your cluster from the IBM® Cloud Private CLI and log in to the Docker private image registry.
   `bx pr login -a https://<cluster_CA_domain>:8443 --skip-ssl-validation`
    Where cluster_CA_domain is the certificate authority (CA) domain. If you did not specify a CA domain, the default value is mycluster.icp. 
4. Install the Helm chart
   `bx pr load-helm-chart --archive <helm_chart_archive> [--clustername <cluster_CA_domain>]`
   Where helm_chart_archive is the name of your compressed Helm chart file and <cluster_CA_domain> is the certificate authority (CA) domain.
5. Update the package repository by using the IBM® Cloud Private cluster management console.
   1. From the IBM® Cloud Private management console, click ***Menu > Manage > Helm Repositories***.
   2. Click Sync Repositories.
   3. In the upper right hand corner of the screen click ***Catalog*** The new Helm charts load into the Catalog, and you can install them into your cluster.

## Assets

Kubernetes Assets in this chart.

**OpenLDAP**
OpenLDAP

see details in [official site](http://www.openldap.org/)

default values below

```
OpenLdap:
  Image: "docker.io/osixia/openldap"
  ImageTag: "1.1.10"
  ImagePullPolicy: "Always"
  Component: "openldap"

  InitImage: "docker.io/busybox"
  InitImageTag: "1.30.1"
  InitImagePullPolicy: "Always"

  Replicas: 1

  Cpu: "512m"
  Memory: "200Mi"

  Domain: "local.io"
  AdminPassword: "admin"
  Https: "false"
  SeedUsers: 
    usergroup: "icpusers"
    userlist: "user1,user2,user3,user4"
    initialPassword: "ChangeMe"
```

**phpLDAPadmin**
LDAP admin UI

see details in [official site](http://phpldapadmin.sourceforge.net/)

default values below
```
PhpLdapAdmin:
  Image: "docker.io/osixia/phpldapadmin"
  ImageTag: "0.7.0"
  ImagePullPolicy: "Always"
  Component: "phpadmin"

  Replicas: 1

  NodePort: 31080

  Cpu: "512m"
  Memory: "200Mi"
```

## Setup IBM Common Services LDAP integration

Detailed information about LDAP support in ICP avilable on the [IBM KnowledgeCenter](https://www.ibm.com/support/knowledgecenter/SSBS6K_2.1.0/user_management/configure_ldap.html)

After the chart is deployed, follow these steps to setup LDAP authentication
 
 1. From the helm release page take note of the OpenLDAP cluster ip and port for your deployment
 2. Navigate to ***Manage > Authentication*** and insert the following details
    #### LDAP Connection
    - Name: `ldap`
    - Type: `Custom`
    - URL: `ldap://<cluster-ip>:389`
    
    #### LDAP authentication
    - Base DN: `dc=local,dc=io` (default value, adjust as needed)
    - Bind DN: `cn=admin,dc=local,dc=io` (default value, adjust as needed)
    - Admin Password: `admin` (default value, adjust as needed)
    
    #### LDAP Filters
    - Group filter: `(&(cn=%v)(objectclass=groupOfUniqueNames))`
    - User filter: `(&(uid=%v)(objectclass=person))`
    - Group ID map: `*:cn`
    - User ID map: `*:uid`
    - Group member ID map: `groupOfUniqueNames:uniquemember`

    Click ***Save***
    
 3. Add users or groups to teams by navigating to ***Manage > Teams***
    - Click ***Create team***
    - Search group or user names to add, and select appropriate roles for each
    

## Credit

Inspired by work done by the [Samsung Cloud Native Computing Team](https://github.com/samsung-cnct) .
