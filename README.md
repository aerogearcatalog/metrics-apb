# Metrics APB

[![](https://img.shields.io/docker/automated/jrottenberg/ffmpeg.svg)](https://hub.docker.com/r/aerogearcatalog/metrics-apb/)
[![Docker Stars](https://img.shields.io/docker/stars/aerogearcatalog/metrics-apb.svg)](https://registry.hub.docker.com/v2/repositories/aerogearcatalog/metrics-apb/stars/count/)
[![Docker pulls](https://img.shields.io/docker/pulls/aerogearcatalog/metrics-apb.svg)](https://registry.hub.docker.com/v2/repositories/aerogearcatalog/metrics-apb/)
[![License](https://img.shields.io/:license-Apache2-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)

## Testing

If you're adding a new changes want you can easily test your changes by simply running `apb test`
The test does the following:
1. Builds the APB 
1. Creates the project in currently targeted OpenShift instance
1. Runs the `provision` role
1. Runs the `test` role which checks that
    * Grafana and Prometheus routes are accessible
    * we're getting successful response from OpenShift auth proxy server

If the test ends successfully, you should see the message `Pod phase Succeeded without returning test results` in your console output.

## How auto-discovery of services work

See the inline comments [here](roles/provision-metrics-apb/templates/prometheus-config-map.yml.j2) and the
example config from Prometheus [here](https://github.com/prometheus/prometheus/blob/master/documentation/examples/prometheus-kubernetes.yml).

## How to Add a New Dashboard while Provisioning a Service

Grafana picks up any dashboard JSON files that are placed inside the `grafana-dashboards-configmap`. This section describes how to add a new dashboard into the `grafana-dashboards-configmap` from inside an APB. The APB that provisions the service should also be responsible for adding the dashboard.

A reusable Ansible role for adding JSON files to a Config Map has been created: https://github.com/aerogear/oc-patch-file-to-configmap-role. The sections below demostrate how to use the role.

#### requirements.yml
Under the `playbooks` directory create a file `requirements.yml`:

```yaml
- src: https://github.com/aerogear/oc-patch-file-to-configmap-role
  version: master
  name: oc-patch-file-to-configmap
```

This file defines the `oc-patch-file-to-configmap` role as a dependency. 

#### Modify Dockerfile
The next step is to modify the default `Dockerfile`. The default APB Dockerfile usually has a list of commands like this:

```
COPY playbooks /opt/apb/actions
COPY roles /opt/ansible/roles
COPY vars /opt/ansible/vars
RUN chmod -R g=u /opt/{ansible,apb}
USER apb
```

Add the following line **before** the `RUN chmod -R g=u /opt/{ansible,apb}` command:


```
RUN ansible-galaxy install -r /opt/apb/actions/requirements.yml -p /opt/ansible/roles
```

This command uses the `ansible-galaxy` CLI tool to install dependencies in `requirements.yml` into the appropriate roles directory. This means when `apb build` is run, the `oc-patch-file-to-configmap` role is installed into the APB image.

#### Usage

It is recommended to use the `oc-patch-file-to-configmap` role directly within the role being used to provision the service (e.g. `provision-keycloak-apb`). The role directory structure should be similar to this:

```bash
├── defaults
│   └── main.yml
├── files
│   ├── ... # role specific files
├── tasks
│   └── main.yml
└── templates
    ├── ... # role specific templates
```

First, place the dashboard json file underneath the `files` directory. e.g. `files/keycloak-dashboard.json`

Next, add the following variables to `defaults/main.yml`

```yaml
# Dashboard config
dashboards_configmap: grafana-dashboards-configmap # this should not change
dashboard_filename: keycloak-dashboard.json # name the file will have inside the config map
dashboard_file_contents: "{{ lookup('file','../files/keycloak-dashboard.json') }}" # Contents of the dashboard file
```

**Note** the `dashboard_filename` is the name the file will have inside the Config Map. **This name should not clash with another service.** If a file with the same name already exists in the Config Map, it will be overwritten.

Lastly, add the following into `tasks/main.yml`

```yaml
- include_role:
    name: oc-patch-file-to-configmap
  vars:
    file_contents: "{{ dashboard_file_contents }}"
    filename: "{{ dashboard_filename }}"
    configmap: "{{ dashboards_configmap }}"
    namespace: "{{ namespace }}"
```

This instructs Ansible to execute the role, passing in the appropriate variables.

**Note** If the usage of this role is still not clear, take a look at the usage in the [Keycloak APB](https://github.com/aerogearcatalog/keycloak-apb).