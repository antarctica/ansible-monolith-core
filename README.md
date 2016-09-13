# Monolith Core (`monolith-core`)

Installs and configures the hosting environment of Monolith Instances, and provides configuration templates to Monolith 
Services.

**This role is NOT part of the BAS Ansible Role Collection (BARC)**

## Overview

This role is used as the core of a [Monolith](https://bitbucket.org/antarctica/monolith) instance, that is 
an instance which will host Monolith services. It is also used by these services, to configure Monolith instances, to 
host such services.

**Note:** This is not a general purpose role, and may not be, and is not designed to be, useful to others. 

This role supports a range of features, some of which are only recommended for specific Monolith environments:

2. setting up [AWS Code Deploy agent](http://docs.aws.amazon.com/codedeploy/latest/userguide/host-cleanup.html) 
  * for Monolith instances
  * installs the AWS Code Deploy agent for managing the deployment of Monolith services
  * see the *AWS Code Deploy agent* sub-section for more information
  * disabled by default
  * **MUST** be used in *production* or *staging* Monolith environment, **MAY** be used in any other environment

3. setting up BAS Service Layer shim
  * for Monolith instances
  * configures a Monolith instance to emulate the minimal viable features of the BAS Service Layer for testing services
  * see the *BAS Service Layer shim* sub-section for more information
  * disabled by default
  * **MUST NOT** be used in *production* or *staging* Monolith environment, **MAY** be used in any other environment

4. configuring a Monolith instance to host a Monolith service
  * for Monolith services
  * configures a Monolith instance to host a Monolith service by generating the relevant configuration files
  * see the *Monolith service Monolith instance configuration* sub-section for more information
  * disabled by default
  * **MAY** be used in any Monolith environment


## Ansible compatibility

* this role supports Ansible 2.x only

## Dependencies

* [**bas-ansible-roles-collection.php7-nginx**](https://galaxy.ansible.com/bas-ansible-roles-collection/php7-nginx/)
  * Minimum version: *0.1.0*

## Requirements

* none

## Usage

### AWS Code Deploy agent

The [AWS Code Deployment agent](http://docs.aws.amazon.com/codedeploy/latest/userguide/host-cleanup.html) is a 
component of the AWS Code Deploy service which polls for new application revisions and deploys them to Monolith 
instances.

The Code Deploy agent is distributed as a Debian package, installing a ruby based application (the agent). In addition 
to installing this package, this role will:

* generate a AWS Code Deploy agent configuration file (i.e. `/etc/codedeploy-agent/conf/codedeployagent.yml`)
* set the `CODEDEPLOY_USER` environment variable, and modify the agent service configuration to use this variable
* if this environment variable is set, the agent can be ran as a non-privileged user, otherwise it will run as root

**Note:** By default the Code Deploy agent is set to run as a conventional `app` non-privileged user. This role will
fail if this user does not exist or the `monolith_core_monolith_non_privileged_user` role variable is not changed.

See the main Monolith project documentation for more information on how AWS Code Deploy is used within Monolith.

### BAS Service Layer shim

Monolith is designed to operate behind a load balancing layer, specifically the 
[BAS Service Layer](https://bitbucket.org/antarctica/menagerie) (Menagerie).

For development Monolith environments this service layer is not used, and means two specific features need to be 
provided by such Monolith environments, specifically:

1. listening to requests on well-known ports (80/443) and passing these to Monolith service ports
2. terminating TLS/SSL requests and passing these with appropriate `forwarded-for` headers, such that Monolith services 
don't need to handle TLS/SSL functions

This role includes a *shim* for the BAS Service Layer which provides these functions:

1. using the Nginx web-server available in the Monolith environment, a reverse proxy is used to forward requests from
well-known ports (80/443) to a single Monolith service port
2. using the Nginx web-server available in the Monolith environment, insecure requests are redirected to a secure 
equivalent, i.e. 'http://' to 'https://'
3. using the Nginx web-server available in the Monolith environment, secure requests are terminated, with the 
appropriate `forwarded-for` headers set

There are numerous limitations with this *shim*:

1. The *shim* provides for a single Monolith service only, i.e. all requests are routed to the same service 
2. The *shim* will not provide any of the other features provided by the BAS Service Layer, such as rate limiting

### Monolith service Monolith instance configuration

Before a Monolith service can be deployed to a Monolith instance (of any environment), the instance must be configured.

This consists of a generating, and enabling, a Nginx server block, which listens for requests on the port assigned to 
the Monolith service.

This role provides the tasks and template necessary to do this, and will be executed where the relevant feature is 
enabled and the relevant role variables are set for the service name, port and document root.

### Typical usage example

To setup a Monolith instance to host a Monolith service (local development or development environments):

```yaml
---

- name: deploy monolith service into a development Monolith instance
  hosts: all
  become: yes
  vars:
    tls_certificate_path: ~/.tls/certs/v.m.tls.crt
    tls_private_key_path: ~/.tls/private/v.m.tls.key
    monolith_core_enable_feature_setup_monolith_feature: true
    monolith_core_enable_feature_menagerie_shim: true
    monolith_core_monolith_service_name: 'example-service'
    monolith_core_monolith_service_port: 4010
    monolith_service_host_name: '_'

  roles:
    - antarctica.monolith-core 

```

### Variables

#### *monolith_core_enable_feature_menagerie_shim*

* **MAY** be specified
* this variable is used as a 'feature flag' for whether tasks for the *shim* for the BAS Service Layer are applied
* see the *BAS Service Layer shim* section for more information
* values **MUST** use one of these options, as determined by Ansible:
    * `true`
    * `false`
* values **SHOULD NOT** be quoted to prevent Ansible coercing values to a string
* default: `false`

#### *monolith_core_enable_feature_aws_code_deploy_agent*

* **MAY** be specified
* this variable is used as a 'feature flag' for whether tasks for installing the AWS Code Deploy agent are applied
* see the *AWS Code Deploy agent* section for more information
* values **MUST** use one of these options, as determined by Ansible:
    * `true`
    * `false`
* values **SHOULD NOT** be quoted to prevent Ansible coercing values to a string
* default: `false`

#### *monolith_core_enable_feature_setup_monolith_service*

* **MAY** be specified
* this variable is used as a 'feature flag' for whether tasks to include a Monolith service in a Monolith instance are 
applied
* see the *Monolith service Monolith instance configuration* section for more information
* values **MUST** use one of these options, as determined by Ansible:
    * `true`
    * `false`
* values **SHOULD NOT** be quoted to prevent Ansible coercing values to a string
* default: `false`

#### *monolith_core_monolith_aws_code_deploy_agent_user*

* **MAY** be specified
* specifies which user the AWS code deploy agent runs as
* values **MUST** be a valid system user, as determined by the operating system
* note: the default value for this variable uses a conventional user used across the Monolith project 
* default: `app`

#### *monolith_core_monolith_service_non_privileged_user*

* **MAY** be specified
* specifies which user is used by Monolith services (i.e. who owns application files)
* values **MUST** be a valid system user, as determined by the operating system
* note: the default value for this variable uses a conventional user used across the Monolith project 
* default: `app`

#### *monolith_core_monolith_service_name*

* **MUST** be specified - this variable has no default value
* specifies the name of a Monolith service, used for naming configuration files, creating a document root, etc.
* example: `example-service`

#### *monolith_core_monolith_service_port*

* **MUST** be specified - this variable has no default value
* specifies the port a Monolith service listens on
* the value for this variable **MUST** be the port assigned to each service, as documented in the main Monolith project
* if developing a proto-type service locally, the Monolith service prototype port, `4011` **MAY** be used
* example: `4010`

#### *monolith_core_monolith_service_host_name*

* **MAY** be specified
* specifies the domain at which a Monolith service is available
* see the main Monolith project for restrictions and guidelines on values which **SHOULD** be used for this variable
* by default, the value of this variable includes the value of the *monolith_core_monolith_service_name* variable
* default: `{{ monolith_core_monolith_service_name }}.web.bas.ac.uk`

#### *monolith_core_monolith_service_root*

* **MAY** be specified
* by default, the value of this variable uses the value of the *monolith_core_monolith_service_name* variable as part 
of a conventional path to deployed applications
* values **MUST** be a valid system path, as determined by the operating system, with suitable permissions
* note the default value for this variable includes setting a prefix of `/public`, this prefix **MUST NOT** be changed
* default: `/srv/apps/{{ monolith_core_monolith_service_name }}/public`

#### *monolith_core_app_aws_codedeploy_agent_desired_version*

* **MAY** be specified
* specifies the version of the AWS Code Deploy agent to install
* values **MUST** be a valid 
[AWS Code Deploy agent version](http://docs.aws.amazon.com/codedeploy/latest/userguide/host-cleanup.html#agent-supported-version), 
as determined by Amazon Web Services
* default: `1.0-1.1011`

#### *monolith_core_app_aws_codedeploy_agent_package_url*

* **MAY** be specified
* specifies the location of the AWS Code Deploy agent Debian package
* values **MUST** be a valid URL, pointing to a package file which can be installed using the `dpkg` utility
* the default value for this variable is a conventional default, and **SHOULD NOT** be changed without good reason
* default: `"https://aws-codedeploy-eu-west-1.s3.amazonaws.com/releases/codedeploy-agent_{{ monolith_core_app_aws_codedeploy_agent_desired_version }}_all.deb"`

## Developing

### Issue tracking

Issues, bugs, improvements, questions, suggestions and other tasks related to this package are managed through the
[BAS Web & Application Team](https://jira.ceh.ac.uk/projects/BASWEB) (BASWEB) project on Jira.

This service is currently only available to BAS or NERC staff, although external collaborators can be added on request.
See our contributing policy for more information.

### Source code

All changes should be committed, via pull request, to the canonical repository:

`ssh://git@stash.ceh.ac.uk:7999/basweb/ansible-monolith-core.git`

A read-only mirror of this repository is maintained on GitHub:

`git@github.com:antarctica/ansible-monolith-core.git`

### Contributing policy

This role welcomes contributions, see `CONTRIBUTING.md` for our general policy.

## Licence

Copyright 2016 NERC BAS.

Unless stated otherwise, all documentation is licensed under the Open Government License - version 3.
All code is licensed under the MIT license.

Copies of these licenses are included within this role.
