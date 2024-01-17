# Installation of Watsonx.ai on OpenShift

This guide provides instructions for a quick PoC kind of installation of Watsonx.ai on OpenShift.<br/>
Watsonx.ai can be installed on public cloud/on-prem - on top of OpenShift. 

## Installation Steps
### Installing Cloud Pak for Data Control Plane

1. Have a running OCP cluster.
   I had 6 worker nodes (m6i.2xlarge) running on AWS of which I allocated 3 nodes for OpenShift Data Foundation(ODF). I installed ODF using Operator Hub.
        
2. Have Podman or docker desktop up and running on your workstation - to pull images from IBM’s registry

3. Install Cloud Pak for Data command-line interface  cpd-cli command line client  on your workstation
  https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=workstation-installing-cloud-pak-data-cli

4. Create an environment variable file
  https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=information-setting-up-installation-environment-variables and source it
   Attached the sample file which I used
       
 5. Login to the OCP cluster using cpd-cli
    
        cpd-cli manage login-to-ocp --username=${OCP_USERNAME} --password=${OCP_PASSWORD} --server=${OCP_URL}
        
 6. Get IBM entitlement key https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=information-obtaining-your-entitlement-api-key by logging into https://myibm.ibm.com/products-services/containerlibrary
    And then 
    Update “pull-secret” secret in “openshift-config” namespace - such that you add
    Or use 
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=cluster-updating-global-image-pull-secret command
    
        cpd-cli manage add-icr-cred-to-global-pull-secret \
        --entitled_registry_key=${IBM_ENTITLEMENT_KEY}


 7.  Create 2 namespaces in OpenShift (cpd-operators, cp4d)
     Which you assigned in the environment variable file
        PROJECT_CPD_INST_OPERATORS=cpd-operators
        PROJECT_CPD_INST_OPERANDS=cp4d
      <img width="1615" alt="image" src="https://user-images.githubusercontent.com/19476054/213525727-e7c31c32-28ba-4cc7-97d6-884bfe136e07.png">

 8. Before you install an instance of IBM Cloud Pak for Data, you must ensure that the project where the operators will be installed can watch the project where the Cloud Pak for Data control plane and services are installed.

    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-applying-required-permissions-projects-namespaces#taskprep-for-cpd-project-permissions__steps

        cpd-cli manage authorize-instance-topology \
        --cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
        --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}

9. Before you install IBM Cloud Pak for Data, you must install the IBM Cloud Pak foundational services Certificate manager and License Service.
    
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=cluster-installing-shared-components#taskshared-components__steps__1

        cpd-cli manage apply-cluster-components \
        --release=${VERSION} \
        --license_acceptance=true \
        --cert_manager_ns=${PROJECT_CERT_MANAGER} \
        --licensing_ns=${PROJECT_LICENSE_SERVICE}
10. Run the cpd-cli manage setup-instance-topology to install IBM Cloud Pak foundational services and create the required ConfigMap: without tethered projects
    
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-installing-cloud-pak-foundational-services


        cpd-cli manage setup-instance-topology \
        --release=${VERSION} \
        --cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
        --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
        --license_acceptance=true \
        --block_storage_class=${STG_CLASS_BLOCK}
11. Review the license for CP4D
    
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-installing-cloud-pak#taskinstall-platform-instance__steps__1 step 2

        cpd-cli manage get-license \
        --release=4.8.1 \
        --license-type=SE

12. Install the operators in the operators project for the instance for Cloud Pak for Data Platform Operator only 
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-installing-cloud-pak#taskinstall-platform-instance__steps__1 step 3

        cpd-cli manage apply-olm \
        --release=${VERSION} \
        --cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
        --components=cpd_platform

13. Install the operands in the operands project for the instance:
    
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-installing-cloud-pak#taskinstall-platform-instance__steps__1 step 4
    For OpenShift Data Foundation storage, to install Cloud Pak for Data control plane only

        cpd-cli manage apply-cr \
        --release=${VERSION} \
        --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
        --components=cpd_platform \
        --block_storage_class=${STG_CLASS_BLOCK} \
        --file_storage_class=${STG_CLASS_FILE} \
        --license_acceptance=true

14. Get the Cloud Pak for data web interface url and credentials
    
    https://www.ibm.com/docs/en/cloud-paks/cp-data/4.8.x?topic=data-installing-cloud-pak#taskinstall-platform-instance__steps__1 step 6

        cpd-cli manage get-cpd-instance-details \
        --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
        --get_admin_initial_credentials=true

### Installing Watsonx.ai service on top of Cloud Pak for Data 
