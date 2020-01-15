# shinyproxy-kubernetes

Docker image for shinyproxy ment to be used with kubernetes.

Most of the work is just picked from [this GitHub repository.](https://github.com/openanalytics/shinyproxy-config-examples/tree/master/03-containerized-kubernetes) This container is to enable customization in more geneal way.

## Using a proxy

Proxy env vars are considered and may just injected using kubernetes env vars.

## application.yml

*application.yml* is the name of the ShinyProxy configuration file. This file has to be injected using a configmap. AS shinyproxy expects the config to be in the same directory as the *shinyproxy.jar* we do create a symbolic link to the subfolder *./config/application.yml* so the config file can be mounted to this directory.

## Example

```yaml

```