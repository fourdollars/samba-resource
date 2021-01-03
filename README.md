 [![GitHub: fourdollars/samba-resource](https://img.shields.io/badge/GitHub-fourdollars%2Fsamba%E2%80%90resource-green.svg)](https://github.com/fourdollars/samba-resource/) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Bash](https://img.shields.io/badge/Language-Bash-red.svg)](https://www.gnu.org/software/bash/) ![Docker](https://github.com/fourdollars/samba-resource/workflows/Docker/badge.svg) [![Docker Pulls](https://img.shields.io/docker/pulls/fourdollars/samba-resource.svg)](https://hub.docker.com/r/fourdollars/samba-resource/)
# samba-resource
[concourse-ci](https://concourse-ci.org/)'s samba-resource

* Inspired by [airtonix/concourse-resource-samba](https://github.com/airtonix/concourse-resource-samba) and [JeffDeCola/resource-template](https://github.com/JeffDeCola/resource-template)

## Config 

### Resource Type

```yaml
resource_types:
- name: resource-samba
  type: registry-image
  source:
    repository: fourdollars/samba-resource
    tag: latest
```

or

```yaml
resource_types:
- name: resource-samba
  type: registry-image
  source:
    repository: ghcr.io/fourdollars/samba-resource
    tag: latest
```

### Resource

* servicename: **required**
* username: optional
* password: optional
* path: optional
* overwrite: optional, true by default and it can be overriden by params.overwrite of put step below.

```yaml
resources:
- name: storage
  type: resource-samba
  source:
    servicename: //domain.name.or.ip/share_point
    username: YourUserName
    password: YourPassWord
    path: PrimaryFolder
    overwrite: false
```

### get step

* path: optional
* files: optional
* skip: optional, set true if you just want to list files and folders.

```yaml
- get: storage
  params:
    path: SecondaryFolder
    files:
      - file1.txt
      - file2.txt
    skip: false
```
```shell
# It acts like the following commands.
$ smbclient //domain.name.or.ip/share_point -U YourUserName YourPassWord -D "PrimaryFolder/SecondaryFolder" -Tc /tmp/backup.tar file1.txt file2.txt
$ cd /tmp
$ tar xf /tmp/backup.tar
$ mv PrimaryFolder/SecondaryFolder/* /tmp/build/get
```

### put step

* from: **required**
* files: optional
* overwrite: optional
* path: optional
* skip: optional if you don't want the [implicit get step](https://concourse-ci.org/jobs.html#put-step) after the put step to download the same content again in order to save the execution time.

```yaml
- put: storage
  params:
    from: SomeFolderInTask
    files:
      - file1.txt
      - file2.txt
    overwrite: false
    path: SecondaryDirectory
  get_params:
    skip: true
```
```shell
# It acts like the following commands.
$ cd /tmp/build/put/SomeFolderInTask
$ tar cf /tmp/backup.tar file1.txt file2.txt
$ smbclient //domain.name.or.ip/share_point -U YourUserName YourPassWord -D "PrimaryFolder/SecondaryFolder" -Tx /tmp/backup.tar
```
