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

```yaml
resources:
- name: storage
  type: resource-samba
  source:
    servicename: //domain.name.or.ip/share_point
    username: YourUserName
    password: YourPassWord
    path: PrimaryFolder
```

### get step

* path: optional

```yaml
- get: storage
  params:
    path: SecondaryFolder
```
```shell
# It acts like the following commands.
$ smbclient //domain.name.or.ip/share_point -U YourUserName YourPassWord -D "PrimaryFolder/SecondaryFolder" -Tc /tmp/backup.tar "*"
$ cd /tmp
$ tar xf /tmp/backup.tar
$ mv PrimaryFolder/SecondaryFolder/* /tmp/build/get
```

### put step

* from: **required**
* files: optional
* path: optional
* skip: optional if you don't want the [implicit get step](https://concourse-ci.org/jobs.html#put-step) after the put step to download the same content again in order to save the execution time.

```yaml
- put: storage
  params:
    from: SomeFolderInTask
    files:
      - file1.txt
      - file2.txt
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
