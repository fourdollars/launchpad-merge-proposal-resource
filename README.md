 [![GitHub: fourdollars/launchpad-merge-proposal-resource](https://img.shields.io/badge/GitHub-fourdollars%2Flaunchpad%E2%80%90merge%E2%80%90proposal%E2%80%90resource-darkgreen.svg)](https://github.com/fourdollars/launchpad-merge-proposal-resource/) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Bash](https://img.shields.io/badge/Language-Bash-red.svg)](https://www.gnu.org/software/bash/) ![Docker](https://github.com/fourdollars/launchpad-merge-proposal-resource/workflows/Docker/badge.svg) [![Docker Pulls](https://img.shields.io/docker/pulls/fourdollars/launchpad-merge-proposal-resource.svg)](https://hub.docker.com/r/fourdollars/launchpad-merge-proposal-resource/)
# launchpad-merge-proposal-resource
[concourse-ci](https://concourse-ci.org/)'s launchpad-merge-proposal-resource to watch the Launchpad merge proposal changes by using https://api.launchpad.net/devel/#branch_merge_proposal

*WIP* *Working In Progress*

## Config 

### Resource Type

```yaml
resource_types:
- name: launchpad-merge-proposal
  type: registry-image
  source:
    repository: fourdollars/launchpad-merge-proposal-resource
    tag: latest
  defaults:
    oauth_consumer_key: test
    oauth_token: csjrGznX4Jq59CB8941N
    oauth_token_secret: wxDNqsCLxzrmhb2K27FRGjc7hdp3zQk0b4N8cnfRzVHnJfCFlHgkGHxDk5qMPTSdQFSsllS4dwGBD18Q
```

or

```yaml
resource_types:
- name: launchpad-merge-proposal
  type: registry-image
  source:
    repository: ghcr.io/fourdollars/launchpad-merge-proposal-resource
    tag: latest
  defaults:
    oauth_consumer_key: test
    oauth_token: csjrGznX4Jq59CB8941N
    oauth_token_secret: wxDNqsCLxzrmhb2K27FRGjc7hdp3zQk0b4N8cnfRzVHnJfCFlHgkGHxDk5qMPTSdQFSsllS4dwGBD18Q
```

### Resource

* oauth_consumer_key: *optional* for private merge proposal, choose what you like.
* oauth_token: *optional* for private merge proposal, run `oauth_consumer_key=what_you_like ./launchpad-api` to get it in ~/.config/launchpad/what_you_like.
* oauth_token_secret: *optional* for private merge proposal, run `oauth_consumer_key=what_you_like ./launchpad-api` to get it in ~/.config/launchpad/what_you_like.
* repo: **required**, a git/bzr repo location such as ~oem-solutions-engineers/pc-enablement/+git/oem-scripts against https://code.launchpad.net/~oem-solutions-engineers/pc-enablement/+git/oem-scripts.
* queue_status: *optional*, to generate the resource version only for the status of "Work in progress", "Needs review" or "Approved"
* debug: *optional*, set "verbose" to enable the verbose debug message.

```yaml
resources:
- name: merge-proposal
  icon: merge
  type: launchpad-merge-proposal
  check_every: 10m
  source:
    repo: ~oem-solutions-engineers/pc-enablement/+git/oem-scripts
    queue_status:
    - Needs review
    - Approved
```

### Example

```yaml
jobs:
- name: check-merge-proposal
  plan:
  - get: merge-proposal
    trigger: true
  - task: check
    config:
      platform: linux
      image_resource:
        type: registry-image
        source:
          repository: alpine
          tag: latest
      inputs:
        - name: merge-proposal
      run:
        path: sh
        args:
        - -ec
        - |
          apk add -q --no-progress jq
          for json in merge-proposal/*.json; do
            jq -r '.web_link + " " + .queue_status' < "$json"
          done
```
