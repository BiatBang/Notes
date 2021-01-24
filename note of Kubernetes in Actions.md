# Kubernetes in Action

## Overview
### Container
Containers make usages of *Linux Namespace* (limits the scope of processes) and *Linux Control Groups* (limits the resource usage of processes) to make the separation. This is why windows suffers from not able to use Docker.
**Terms for docker**
- Image: a packaged program with metadata (env info).
- Registry: docker's repo. push to registry or pull from registry. 
- Container: a linux container, with limited resource.
