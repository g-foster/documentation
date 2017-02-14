---
title: How do I use Composer with GitHub personal access token?
menu:
  main:
    parent: 'faq'
    weight: 40
weight: 40
linkTitle: Setup GitHub Token
---
If you need to download private repositories from GitHub during the build
process, an access token has to be available during `composer install`.

To set up that, make sure you are using one of the Docker images from
[ContinuousPipe dockerfiles](https://github.com/continuouspipe/dockerfiles)
as base image. These images are prepared to take a GITHUB_TOKEN 
build argument which is used to set the token like this:
`composer global config github-oauth.github.com "$GITHUB_TOKEN"`

{{< warning title="Warning" >}}
Token can be added as either a Docker build argument or an environment 
variable. When it is passed as a build argument, the token will be visible
in the image history (see `docker history <image-name>` command). When it 
is given as an environment variable, the token is added to the image 
itself (see `docker inspect --format '{{ index (index .Config.Env) }}' <image-name>`
).
However the composer call above will save it in the filesystem of the image.
So the final Docker images should be pushed into a private repository.
{{< /warning >}}

Let's suppose you have a service called `api` which requires an access 
token.
Open the flow configuration and add a new variable named `GITHUB_TOKEN`.
After that update the YAML file to pass it as an environment variable:
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

When you build the image locally with Docker Compose, then create an environment
variable in your shell like `export GITHUB_TOKEN=...`. See the 
[Args section](https://docs.docker.com/compose/compose-file/#args) of Docker Compose
file reference for more details.