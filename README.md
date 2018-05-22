chart-dropoff
===
Dropping off files, binaries to host for kubernetes cluster using Helm chart.

This chart will create DaemonSet that will facilitate dropping off any files to the host.

To use the chart you will need to create a docker image that runs a script that will copy your file to the host mounted path. 

Docker Image Build
---
You will build your image to your own specification. Then add a bash script to the `ENTRYPOINT`. For example to drop a binary:

**installer.sh**

    #!/bin/sh

    set -o errexit
    set -o pipefail

    PKG=your-binary

    cp "/${PKG}" "/docker-mnt/.${PKG}"
    mv -f "/docker-mnt/.${PKG}" "/docker-mnt/${PKG}"
    chmod +x "/docker-mmnt/${PKG}"

    while : ; do
      sleep 30
    done

**Dockerfile**

    FROM busybox

    ADD your-binary /
    ADD installer.sh /
    RUN mkdir /docker-mmnt
    VOLUME /docker-mmnt

    ENTRYPOINT ["sh", "/installer.sh"]

Values 
---

|         Name        |    Requirement    |        Default       |                 Description             |
|:--------------------|:-----------------:|:--------------------:|:----------------------------------------|
| pathInImage | mandatory |    | The path in the pod that will mount to host path, for example the `VOLUME` name in your Dockerfile |
| hostPath | mandatory |    | The destination of the drop on host. For example the path your binary will be dropped into |
| repository | mandatory |  | Docker image reposository |
| tag | optional | latest  | Image tag |
| pullSecret | optional |  | Image pull secrect if using private repo |
| pullPolicy | optional | Always  | Image pull policy |
| component | optional |  | DaemonSet label. Useful when using this chart for dropping different files |


Installation
---
    
    $ helm install --namespace <your-namespace> . --set <name>=<value>,<parent.children>=<value>

Upgrade
---
    
    $ helm upgrade <release-name>

Delete
---

    $ helm delete <release-name>
