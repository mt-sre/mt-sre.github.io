---
title: "SLO Dashboards"
date: 2022-12-15T00:53:51+01:00
---

Development teams are required to co-maintain, in conjunction with
the MT-SRE Team, SLO Dashboards for the Addons they develop. This
document explains how to bootstrap the dashboard creation and
deployment.

## First Dashboard

* Fork/clone the [managed-tenants-slos repository](https://gitlab.cee.redhat.com/service/managed-tenants-slos).
* Create the following directory structure:

```yaml
├── <addon-name>
│   ├── dashboards
│   │   └── <addon-name>-slo-dashboard.configmap.yaml
│   └── OWNERS
.
```

Example `OWNERS`:

```yaml
approvers:
- akonarde
- asegundo
```

`<addon-name>-slo-dashboard.configmap.yaml` contents (replace all occurrences of `<addon-name>`):

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <addon-name>-slo-dashboard
  labels:
    grafana_dashboard: "true"
  annotations:
    grafana-folder: /grafana-dashboard-definitions/Addons
data:
  mtsre-rhods-slos.json: |
    {
      "annotations": {
        "list": [
          {
            "builtIn": 1,
            "datasource": {
              "type": "grafana",
              "uid": "-- Grafana --"
            },
            "enable": true,
            "hide": true,
            "iconColor": "rgba(0, 211, 255, 1)",
            "name": "Annotations & Alerts",
            "target": {
              "limit": 100,
              "matchAny": false,
              "tags": [],
              "type": "dashboard"
            },
            "type": "dashboard"
          }
        ]
      },
      "editable": true,
      "fiscalYearStartMonth": 0,
      "graphTooltip": 0,
      "links": [],
      "liveNow": false,
      "panels": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "4rNsqZfnz"
          },
          "fieldConfig": {
            "defaults": {
              "color": {
                "mode": "thresholds"
              },
              "custom": {
                "align": "auto",
                "displayMode": "auto",
                "inspect": false
              },
              "mappings": [],
              "thresholds": {
                "mode": "absolute",
                "steps": [
                  {
                    "color": "green",
                    "value": null
                  },
                  {
                    "color": "red",
                    "value": 80
                  }
                ]
              }
            },
            "overrides": []
          },
          "gridPos": {
            "h": 16,
            "w": 3,
            "x": 0,
            "y": 0
          },
          "id": 2,
          "options": {
            "footer": {
              "fields": "",
              "reducer": [
                "sum"
              ],
              "show": false
            },
            "showHeader": true
          },
          "pluginVersion": "9.0.1",
          "targets": [
            {
              "datasource": {
                "type": "prometheus",
                "uid": "4rNsqZfnz"
              },
              "editorMode": "code",
              "expr": "group by (_id) (subscription_sync_total{name=\"${addon_name}\"})",
              "format": "table",
              "range": true,
              "refId": "A"
            }
          ],
          "title": "Clusters",
          "transformations": [
            {
              "id": "groupBy",
              "options": {
                "fields": {
                  "_id": {
                    "aggregations": [],
                    "operation": "groupby"
                  }
                }
              }
            }
          ],
          "type": "table"
        }
      ],
      "schemaVersion": 36,
      "style": "dark",
      "tags": [],
      "templating": {
        "list": [
          {
            "hide": 2,
            "name": "addon_name",
            "query": "addon-<addon-name>",
            "skipUrlSync": false,
            "type": "constant"
          }
        ]
      },
      "time": {
        "from": "now-6h",
        "to": "now"
      },
      "timepicker": {},
      "timezone": "",
      "title": "<addon-name> - SLO Dashboard",
      "version": 0,
      "weekStart": ""
    }
```

* Create a Merge Request adding the files to the managed-tenants-slos git repository.
* Ping @mt-sre-ic in the #forum-managed-tenants Slack channel for review.

## Dashboard Deployment

Merging of the above merge request is a prerequisite for this step.

The dashboard deployment happens through app-interface, using saas-files.

* For each new Addon, we need to create a new saas-file in app-interface.
* Give ownership of the saas-file to your team using an app-interface role file.

Example Merge Request content to app-interface:

<https://gitlab.cee.redhat.com/service/app-interface/-/commit/9306800aabaca18cd034dfb3933a12d29506fa08>

* Ping `@mt-sre-ic` in the `#forum-managed-tenants` Slack channel for approval.
* Merge Requests to app-interface are constantly reviewed/merged by AppSRE.
  After the MT-SRE approval, wait until the Merge Request is merged.

## Accessing the Dashboards

Once the app-interface merge request is merged, you will see your ConfigMaps
being deployed in the `#sd-mt-sre-info` Slack channel. For example:

```bash
[app-sre-stage-01] ConfigMap odf-ms-cluster-status applied
...
[app-sre-prod-01] ConfigMap odf-ms-cluster-status applied
```

Once the dashboards are deployed, you can see them here:

* STAGE: <https://grafana.stage.devshift.net/dashboards/f/aGqy3WB7k/addons>
* PRODUCTION: <https://grafana.app-sre.devshift.net/dashboards/f/sDiLLtgVz/addons>

## Development Flow

After all the configuration is in place:

STAGE:

* Dashboards on the STAGE Grafana instance should not be used by
  external audiences other than the people developing the dashboards.
* Changes in the `managed-tenants-slos` repository can be merged by the
  development team with "/lgtm" comments from those in the OWNERS file.
* After merged, changes are automatically delivered to the STAGE grafana instance.

PRODUCTION:

* The dashboards on the PRODUCTION Grafana are pinpointed to a specific git
  commit from the managed-tenants-slos repository in the corresponding
  saas-file in app-interface.
* After patching the git commit in the saas-file, owners of the saas-file can merge the promotion
  with a "/lgtm" comment in the app-interface Merge Request.
