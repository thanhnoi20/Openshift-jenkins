This project is cloned from https://github.com/cloudowski/jenkins-openshift-pipeline.git


# Jenkins on OpenShift

This repo contains resources used to build a pipeline for building new Jenkins
container image for OpenShift.

# Building a new image - step by step

* Create a new project

  ```
  oc new-project jenkins-builder
  ```

* For OKD/Minishift import jenkins-persistent template - for some reasons it's
  missing in newer version (skip it if your cluster already has it)

  ```
  oc apply -f https://raw.githubusercontent.com/openshift/origin/master/examples/jenkins/jenkins-persistent-template.json -n openshift
  ```

* Start a new Jenkins instance

  ```
  oc process -n openshift jenkins-persistent -p MEMORY_LIMIT=1024M|oc apply -f- -n jenkins-builder
  oc set env dc/jenkins \
  OVERRIDE_PV_CONFIG_WITH_IMAGE_CONFIG=true \
  OVERRIDE_PV_PLUGINS_WITH_IMAGE_PLUGINS=true \
  CASC_JENKINS_CONFIG=/var/lib/jenkins/jenkins-casc.yaml
  ```

* Assign an admin role to jenkins

  ```
  oc adm policy add-cluster-role-to-user admin -z jenkins
  ```

*  Create a pipeline

   ```
   oc process -f jenkins-pipeline-template.yaml|oc apply -f- -n jenkins-builder
   ```

* Run the pipeline job

  ```
  oc start-build build-image-jenkins-master
  ```

  Log in to Jenkins and check the build progress. Approve its promotion to **openshift** project to make it available to other users on your cluster.

