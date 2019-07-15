OptaPlanner Workshop Module 1
===
This is `module 1` of a full-day hands-on OptaPlanner / Red Hat Business Optimizer workshop. Business Optimizer is the A.I. Constraint Solver technology of Red Hat Decision Manager.

This workshop provides developers and architects an introduction to planning problems, A.I. constraints solvers, and more spefically, how OptaPlanner can find optimal solutions to these kind of problems. We'll demonstrate how OptaPlanner's solutions to these kind of problems is far superiour than solutions created by humans or traditional algorithms, and how these solutions can bring a strategical advantage to its users.

Agenda
===
* What is a planning problem?
* Use case overview
* Introduction to OptaPlanner
* Domain Modeling concepts
* Persistence
* Solver Configuration


Lab Instructions on OpenShift
===

Note that if you have installed the lab infra via APB, the lab instructions are already deployed.

Here is an example Ansible playbook to deploy the lab instruction to your OpenShift cluster manually.
```
- name: Create Guides Module 1
  hosts: localhost
  tasks:
  - import_role:
      name: siamaksade.openshift_workshopper
    vars:
      project_name: "guide-m1"
      workshopper_name: "OptaPlanner Workshop V1 Module-1"
      project_suffix: "-XX"
      workshopper_content_url_prefix: https://raw.githubusercontent.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m1-guides/master
      workshopper_workshop_urls: https://raw.githubusercontent.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m1-guides/master/_optaplanner-workshop-module1.yml
      workshopper_env_vars:
        PROJECT_SUFFIX: "-XX"
        COOLSTORE_PROJECT: coolstore{{ project_suffix }}
        OPENSHIFT_CONSOLE_URL: "https://master.seoul-2922.openshiftworkshop.com"
        ECLIPSE_CHE_URL: "http://che-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        GIT_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        NEXUS_URL: "http://nexus-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        LABS_DOWNLOAD_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"

      openshift_cli: "/Users/doh/cloud-native-app-dev/oc --server https://master.seoul-2922.openshiftworkshop.com"
```
