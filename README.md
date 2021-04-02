Google Compute Engine (GCE)
=========

An Ansible role ables to manage GCE resources and simplify the integration process in other Ansible projects that require Google Cloud support.

Requirements
------------

* python3
* ansible >= 2.9
* requests >= 2.18.4
* google-auth >= 1.3.0

Role Variables
--------------

You can manage (create / update / remove) GCE resources through role variables instead of writing tasks.

Defaults variables are include in `defaults/main.yml`, you can create your custom version in the Ansible project that includes this role.

Display GCP debug info
```
gcp_debug: yes
```

GCP Project configuration is included as environment variables:

* `GCP_PROJECT_ID`
  The ID of your Google Project
* `GCP_SERVICE_ACCOUNT_FILE`
  Path of your service account files

Before running the Playbook you should set environment variables
```
export GCP_PROJECT_ID=my-project-id
export GCP_SERVICE_ACCOUNT_FILE=/path/to/my/service-account.json
```

Alternatively, you could override this behavior and define a custom project id and service account file using Ansible variables

```
gcp_project_id: "my-project-if"
gcp_service_account_file: "{{ lookup('env','GCP_SERVICE_ACCOUNT_FILE') }}"
```
To create a new service account please refer to official Google documentation:  
https://cloud.google.com/iam/docs/creating-managing-service-accounts

**Instance Configuration**
The definition below configure a new GCP instance with all required properties.  
You must specify an available service account to assign to the instance, usually there is a default service account for GCP instances.  
Please check [IAM accounts in Google Cloud Console](https://console.cloud.google.com/iam-admin/serviceaccounts), default service account has a pattern like `000000000000-compute@developer.gserviceaccount.com`.

You should also define **Google API permissions** for the new instance, a partial list of available access scopes is displayed at the of this section.

Example access scopes include:
* https://www.googleapis.com/auth/compute. Full control access to Compute Engine methods.
* https://www.googleapis.com/auth/compute.readonly. Read-only access to Compute Engine methods.
* https://www.googleapis.com/auth/devstorage.read_only. Read-only access to Cloud Storage.
* https://www.googleapis.com/auth/logging.write. Write access to the Compute Engine logs.

All GCE instance properties are mandatory, labels as well

```
gce_instance:
  name: "my-ansible-instance"
  zone: "europe-west1-b"
  state: present
  type: n1-standard-1
  delete_protection: no
  service_account: "000000000000-compute@developer.gserviceaccount.com"
  api_auth:
    - "{{ gcp_api_scopes.STORAGE_READ_WRITE }}"
  labels:
    team: devops
    application: ansible
    env: production

```

**Disks Configuration**
You can chose 
```
gce_disk:
  name: "my-ansible-disk"
  zone: "europe-west1-b"
  device:
    d0:
      size: 50
      type: pd-standard
      image: projects/debian-cloud/global/images/debian-10-buster-v20200910
      state: present
      labels:
        team: devops
        application: awx
        env: production
    d1:
       size: 50
       type: pd-standard
       state: present
       labels:
         team: devops
         application: awx
         env: production
```

**VPC Network**
```
gce_network:
  n0:
    name: default
    create_subnet: no
    state: present
```

**External IP Address**
```
gce_ip:
  name: "{{ gce_resource_name }}"
  state: present
  region: "{{ gce_resource_region }}"
```

**Google API Scopes**
```
gcp_api_scopes:
  COMPUTE: https://www.googleapis.com/auth/compute
  MONITORING: https://www.googleapis.com/auth/monitoring
  LOGGING_WRITE: https://www.googleapis.com/auth/logging.write
  MONITORING_WRITE: https://www.googleapis.com/auth/monitoring.write
  SERVICECONTROL: https://www.googleapis.com/auth/servicecontrol
  SERVICE_MANAGEMENT_READONLY: https://www.googleapis.com/auth/service.management.readonly
  TRACE_APPEND: https://www.googleapis.com/auth/trace.append
  STORAGE_READ_WRITE: https://www.googleapis.com/auth/devstorage.read_write
```

Run in a Playbook
----------------

Create a new Playbook and include the role and your custom configuration file (check `defaults/main.yml`):

## Basic Project Tree

```
ansilbe-project
├── create.yml
├── roles
│   └── gce
│       ├── defaults
│       │   └── main.yml
│       ├── handlers
│       ├── meta
│       ├── README.md
│       ├── tasks
│       ├── tests
│       └── vars
└── vars
    └── gce-config.yml
```

In the example below an example of `create.yml` Playbook:

```yaml
# Build new disk image (template) for AI machines provisioning
- name: Create a new GCE instance
  hosts: localhost
  gather_facts: no
  vars_files:
    - vars/gce-config.yml

  roles:
    - role: ansible-role-gce
```

It will create a new instance as defined in `vars/gce-config.yml`


License
-------

MIT

Author Information
------------------

Fabio Ferrari - Cloud Architect and DevOps Engineer  
https://particles.io

