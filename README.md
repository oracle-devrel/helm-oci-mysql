# oci-mysql-db-system-helm


## Introduction

The OCI MySQL Database System Helm Chart makes it easy to create and manage MySQL Database Systems (MDS) from a Kubernetes cluster deployed within Oracle CLoud Infrastrcuture (OCI). The Kubernetes cluster can be a cluster deployed using Oracle Container Engine for Kuberntes (OKE), or it can be a customer managed cluster deployed on virtual machine instances. 

This Helm chart relies on the OCI Service Operator for Kubernetes (OSOK) and it is a pre-requisite to have OSOK deployed within the cluster to use this Helm chart.


## Pre-requisites

- A Kuberntes cluster deployed in OCI 
- [OCI Service Operator for Kuberntes (OSOK) deployed in the cluster](https://github.com/oracle/oci-service-operator/blob/main/docs/installation.md)
- [kubectl installed and using the context for the Kubernetes cluster where the MDS resource will be deployed](https://kubernetes.io/docs/tasks/tools/)
- [Helm installed](https://helm.sh/docs/intro/install/)


## Installation of Helm chart

**1. Clone or download the contents of this repo** 
     
     git clone https://github.com/chiphwang1/oci-mysql-db-system-helm.git

**2. Change to the directory that holds the Helm Chart** 

      cd ./oci-mysql-db-system-helm

**3. Populate the values.yaml file with information to deploy the MDS resource**


**4. Create the namespace where the MDS resource will be deployed**

     kubectl create ns <namespace name>

**5. Install the Helm chart. Best practice is to assign the username and password during the installation of the Helm chart instead of adding it to the values.yam file.**

     helm -n <namespace name> install \
     --set database.username=<database password> \  
     --set database.password=<database password> \
       <name for this install> .
  
  **Example:**
  helm -n mysqldb --set database.username=admin  --set database.password=Admin12345 ocimds .
  
 The password must be between 8 and 32 characters long, and must contain at least 1 numeric character, 1 lowercase character, 1 uppercase character, and 1   special (nonalphanumeric) character.


**6. List the Helm installation**

     helm  -n <namespace name> ls

**7. To uninstall the Helm chart**

     helm uninstall -n <namespace name> <name of the install> .
     
  **Note:**
 uninstalling the helm chart will only remove the mysqldbsystem resource from the cluster and not from OCI. You will need to use the console or the OCI cli to remove the MDS from OCI. This is to prevent accidental deletion of the database.

## MySQL DB System value.yaml Specification


| Parameter                          | Description                                                         | Type   | Mandatory |
| ---------------------------------- | ------------------------------------------------------------------- | ------ | --------- |
| `name` | name of MySQL DB System resource. This is name that is used in the Kubernetes cluster | string | yes  |
| `spec.displayName` | The user-friendly name for the Mysql DbSystem. The name does not have to be unique. | string | yes     |
| `spec.compartmentId` | The [OCID](https://docs.cloud.oracle.com/Content/General/Concepts/identifiers.htm) of the compartment of the Mysql DbSystem. | string | yes       |
| `spec.shapeName` | The name of the shape. The shape determines the resources allocated  | string | yes       |
| `spec.subnetId` | The OCID of the subnet the DB System is associated with.  | string | yes       |
| `spec.dataStorageSizeInGBs`| Initial size of the data volume in GBs that will be created and attached. Keep in mind that this only specifies the size of the database data volume, the log volume for the database will be scaled appropriately with its shape. | int    | yes       |
| `spec.isHighlyAvailable` | Specifies if the DB System is highly available.  | boolean | yes       |
| `spec.availabilityDomain`| The availability domain on which to deploy the Read/Write endpoint. This defines the preferred primary instance. | string | yes        |
| `spec.faultDomain`| The fault domain on which to deploy the Read/Write endpoint. This defines the preferred primary instance. | string | no        |
| `spec.configuration.id` | The OCID of the Configuration to be used for this DB System. [More info about Configurations](https://docs.oracle.com/en-us/iaas/mysql-database/doc/db-systems.html#GUID-E2A83218-9700-4A49-B55D-987867D81871)| string | yes |
| `spec.description` | User-provided data about the DB System. | string | no |
| `spec.hostnameLabel` | The hostname for the primary endpoint of the DB System. Used for DNS. | string | no |
| `spec.mysqlVersion` | The specific MySQL version identifier. | string | no |
| `spec.port` | The port for primary endpoint of the DB System to listen on. | int | no |
| `spec.portX` | The TCP network port on which X Plugin listens for connections. This is the X Plugin equivalent of port. | int | no |
| `spec.ipAddress` | The IP address the DB System is configured to listen on. A private IP address of your choice to assign to the primary endpoint of the DB System. Must be an available IP address within the subnet's CIDR. If you don't specify a value, Oracle automatically assigns a private IP address from the subnet. This should be a "dotted-quad" style IPv4 address. | string | no |
| `database.username` | The admin username for the administrative user for the MuSQL DB Systesm. This should be assigned during the deployment of the Helm chart and not kept in the values.yaml file| string | yes       |
| `database.password` | The admin password for Mysql DB System. The password must be between 8 and 32 characters long, and must contain at least 1 numeric character, 1 lowercase character, 1 uppercase character, and 1 special (nonalphanumeric) character. | string | yes       |
| `autoDB.enabled` | set to true to automatically create a database in the Mysql DB System. Leave this field empty otherwise. | string | no       |
| `DBName` | if autoDB.enabled is set to true, name the database in this field. | string | no       |



## Useful commands 


**1. To check the status of the MySQL DB System run the following command**
     
     kubectl -n <namespace of mysqldbsystem> get mysqldbsystems -o wide

**2. To describe the MySQL DB System run the following command** 
     
     kubectl -n <namespace of mysqldbsystem> describe mysqldbsystems <name of mysqldbsystem>

**3. To retreive the OCID of MySQL DB System run the following command** 

      kubectl -n <namespace of mysqldbsystem> get mysqldbsystems <name of mysqldbsystem> -o jsonpath="{.items[0].status.status.ocid}"
 
**4. To retrive the admin username of the MySQL DB System run the following command**
     
     kubectl -n  <namespace of mysqldbsystem>  get secret mysqlsecret -o  jsonpath="{.data.username}" | base64 --decode

**5. To retrive the admin password of the MySQL DB System run the following command**
     
     kubectl -n  <namespace of mysqldbsystem>  get secret mysqlsecret -o  jsonpath="{.data.password}" | base64 --decode

**6. To retreive endpoint information for the MySQL DB System run the following command**
     
     kubectl -n <namespace of mysqldbsystem> get secret <name of the MySQL DB System>  -o jsonpath='{.data.Endpoints}' | base64 --decode



## Additional Resources

- [OCI Service Operator for Kuberntes (OSOK) deployed in the cluster](https://github.com/oracle/oci-service-operator)
- [MySQL SB System Service](https://www.oracle.com/mysql/)

