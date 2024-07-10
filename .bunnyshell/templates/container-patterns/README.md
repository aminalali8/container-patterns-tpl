# Container Patterns Template Documentation

## Overview
The `container-patterns-tpl` template demonstrates how to utilize init and sidecar containers in Bunnyshell. It provides practical examples to illustrate their setup and interaction within a containerized environment.

## Init Container
The init container is responsible for executing preliminary tasks before the main application starts. In this example, the init container creates a file that can be accessed by the sidecarcontainer once VS Code is deployed.

### Init Container Definition
To define an init container within the main container's configuration, you use the `init_containers` field. Here's the configuration snippet:

```yaml
# Within the definition of the main container bind the init container
pod:
  init_containers: 
    - 
      from: init-container # this is the name of the component that has the kind InitContainer
      name: init-code-server
      shared_paths: # Here you define the paths you need shared with the main container
        - 
          # mount destination
          path: /tmp 
          target:
            # mount source
            path: /usr/src/app 
            # container in which the path above is located ('@parent' here is the container where the 
            # init container is bound to, i.e., the frontend in this case)
            container: '@parent' 
          # target is the main container, alternatively self would reflect that the content should be the content 
          # within the init-container
          initial_contents: '@target' 
```

### Init Container Implementation
The init container uses the `busybox` image to create a file in the specified paths. Here is the definition of the init container component:

```yaml
- 
  kind: InitContainer
  name: init-container
  dockerCompose:
    image: 'busybox:latest'
    command: ['sh', '-c', 'echo "Hello, World!" > /tmp/init-container.txt']
```

## Sidecar Container
The sidecar container runs concurrently with the main application, providing auxiliary functions. In this template, the sidecar container sets up a VS Code web server and mounts the source code from the frontend to the workspace folder in the VS Code container.

### Sidecar Container Definition
Similar to the init container, the sidecar container is defined within the main container's configuration using the `sidecar_containers` field:

```yaml
- 
  kind: frontend
  ...
  pod: 
    sidecar_containers:
      -
        from: code-server
        name: sidecar-code-server
        shared_paths:
          -
            path: /cs_workspace
            target:
              path: /usr/src/app
              container: '@parent'
            initial_contents: '@target'
```

### Sidecar Container Implementation
The sidecar container utilizes the `code-server` image, populates environment variables via template variables, exposes the port for the VS Code web interface, and attaches a volume to persist the configuration. Here is the full definition of the sidecar container component:

```yaml
- 
  kind: SidecarContainer
  name: code-server
  dockerCompose:
    image: 'lscr.io/linuxserver/code-server:4.90.3'
    environment:
      DEFAULT_WORKSPACE: /cs_workspace
      HASHED_PASSWORD: '{{template.vars.HASHED_PASSWORD}}'
      PASSWORD: '{{template.vars.PASSWORD}}'
      PGID: '{{template.vars.PGID}}'
      PROXY_DOMAIN: '{{template.vars.PROXY_DOMAIN}}'
      PUID: '{{template.vars.PUID}}'
      SUDO_PASSWORD: '{{template.vars.SUDO_PASSWORD}}'
      SUDO_PASSWORD_HASH: '{{template.vars.SUDO_PASSWORD_HASH}}'
      TZ: '{{template.vars.TZ}}'
    ports:
      - '8443:8443'
  volumes:
    -
      name: code-server-data
      mount: /config
      subPath: ''
```

## Summary
This template showcases how to effectively use init and sidecar containers within a Bunnyshell environment:
- **Init Container**: Performs preliminary setup tasks, such as creating necessary files before the main application starts.
- **Sidecar Container**: Runs alongside the main application, providing additional services, like a web-based VS Code server.

By using these container patterns, you can modularize your application setup and improve the maintainability and scalability of your containerized solutions.