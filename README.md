# docker-clamav
![ClamAV Logo](https://www.clamav.net/assets/clamav-trademark.png)

![ClamAV latest.stable](https://img.shields.io/badge/ClamAV-latest.stable-brightgreen.svg?style=flat-square)

Dockerized open source antivirus daemons for use with 
- to use it via a [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) proxy like [@solita](https://github.com/solita) made [clamav-rest](https://github.com/solita/clamav-rest) or
- to directly connect to *clamav* via TCP port `3310`

ClamAV daemon as a Docker image. It *builds* with a current virus database and
*runs* `freshclam` in the background constantly updating the virus signature database. `clamd` itself
is listening on exposed port `3310`.

# Credits
This is a fork of [https://github.com/mko-x/docker-clamav](https://github.com/mko-x/docker-clamav). The main difference is that this is a very simplified version with only one tag based on alpine. This also adds back automatic helath check.

## Usage
```bash
    docker run -d -p 3310:3310 shaunakv1/docker-clamav
```

Linked usage recommended, to not expose the port to "everyone".
```bash
    docker run -d --name av shaunakv1/docker-clamav
    docker run -d --link av:av application-with-clamdscan-or-something
```
## Azure Container Instances Deployment

1. Create a deploymet spec file `clamav.yaml` like below
    ```yaml
    apiVersion: "2019-12-01"
    location: eastus2
    name: web-clamav-runtime
    properties:
    containers:
        - name: web-clamav-runtime
        properties:
            environmentVariables: []
            image: mkodockx/docker-clamav:alpine
            ports:
            - port: 3310
            resources:
            requests:
                cpu: 2.0
                memoryInGB: 4
            livenessProbe:
            exec:
                command:
                - "./check.sh"
            initialDelaySeconds: 60
            timeoutSeconds: 120
            periodSeconds: 60
    osType: Linux
    restartPolicy: OnFailure
    ipAddress:
        type: Public
        ports:
        - protocol: tcp
            port: 3310
        dnsNameLabel: ocm-web-clamav
    tags: {}
    type: Microsoft.ContainerInstance/containerGroups
    ```

3. Login using az cli
    ```
        $> az login
    ```
4. Deploy Azure container instance
    ```
        $> az container create --resource-group <resource-group-name> --file clamav_aci.yaml
    ```
5. Stream Logs to make sure it works
    ```
        $> az container logs --follow --resource-group <resource-group-name> --name web-clamav-runtime
    ```
6. To delete the container service use
    ```
        $> az container delete --resource-group <resource-group-name> --name web-clamav-runtime
    ```

## Persistency
Virus update definitions are stored in `/var/lib/clamav`. To store the defintion just mount the directory as a volume, `docker run -d -p 3310:3310 -v ./clamav:/var/lib/clamav mkodockx/docker-clamav:latest`

