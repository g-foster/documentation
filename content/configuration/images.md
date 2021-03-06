---
title: Building the Docker Images
menu:
  main:
    parent: 'configuration'
    weight: 35

weight: 35
---
## The YAML bit

Before deploying anything, you will most of the time want ContinuousPipe to build your Docker images. Being probably the simple task, the only mandatory parameter being the image name you want to build.

``` yaml
tasks:
    images:
        build:
            services:
                web:
                    image: docker.io/your-namespace/your-application
```

You can obviously use any other image name as long as it contains the Docker registry as first part of it. You can also see that it's configured per _service_, here we have only one (named `web`) that we need to deploy.

ContinuousPipe is guessing parameters from your `docker-compose.yml` file. The `image` is the only required value if you have such a Docker Compose file.

## Naming strategy
At the moment, there are two naming strategies: the default one being the `sha1` strategy, that basically builds a tag per commit SHA1. If you require it you can use the `branch` strategy that will create a tag per branch.

``` yaml
tasks:
    images:
        build:
            services:
                web:
                    # ...
                    naming_strategy: branch
```

{{< warning title="Warning" >}}
we do not recommend the branch naming strategy as when updated, some clusters might not force pull the new image while rolling-updating the services.
{{< /warning >}}

## Arguments

If you need to inject token or strings in your build process in order to download private dependencies for instance, you can use [Docker build arguments](https://docs.docker.com/engine/reference/builder/#/arg).

The following example shows how to be able to install PHP private dependencies (using [Composer](http://getcomposer.org/)) using a private GitHub token:

``` yaml
# Dockerfile

# ...

ARG GITHUB_TOKEN=

RUN composer config github-oauth.github.com $GITHUB_TOKEN && \
    composer install -o --no-interaction && \
    composer clear-cache

# continuous-pipe.yml

tasks:
    images:
        build:
            environment:
                - name: GITHUB_TOKEN
                  value: ${GITHUB_TOKEN}

            # ...
```
Note: this assume that you have defined the `GITHUB_TOKEN` variable somewhere. You can refer to the [variable section]({{< relref "configuration-files.md#variables" >}}).

If you want to build multiple services at the same time, you can also provide the build argument per service:

``` yaml
# continuous-pipe.yml
tasks:
    images:
        build:
            services:
                api:
                    environment:
                        - name: GITHUB_TOKEN
                          value: ${GITHUB_TOKEN}

            # ...
```

## Artifacts

In order to build small images and/or hide secret values requried during the build process, you can use artifacts. These artifacts will allow you to share files or folders between different build steps that use a combination of Dockerfiles, context and build arguments.

![](/images/multi-step-building.png)

``` yaml
# continuous-pipe.yml
tasks:
    images:
        build:
            services:
                ui:
                    steps:
                        - docker_file_path: ./Dockerfile-build
                          environment:
                            - name: GITHUB_TOKEN
                              value: ${GITHUB_TOKEN}
                          write_artifacts:
                            - name: dist
                              path: /dist
                        - docker_file_path: ./Dockerfile
                          read_artifacts:
                            - name: dist
                              path: /dist

    # ...
```


