# Docker Stack Deploy Demo

This repository contains a set of files which can be used to reproduce a problem where the `docker stack deploy` command does not correctly orchestrate the rollout of a service update when the `com.docker.ucp.mesh.http` label is present on a service.

## Steps to reproduce the problem

1. Source a UCP client bundle
2. Deploy a stack using the stack files:
    ```sh
    docker stack deploy -c docker-stack.yml -c docker-stack-black.yml demo
    ```
    The containers should take 10 seconds to become healthy.
3. Start a second shell session, source your UCP client bundle again and run the following to start watching the containers involved in the stack above:
    ```sh
    watch docker container ls -f name=demo_nginx
    ```
4. Run another deployment using the opposite stack file (e.g. `red` vs `black`):
    ```sh
    docker stack deploy -c docker-stack.yml -c docker-stack-red.yml demo
    ```
5. Switch back to the tab where the watch command is executing.  You will see both replicas of the service get stopped at the same time instead of deploying the replicas one at a time and waiting for the health checks like it is supposed to.
6. Open `docker-stack.yml`, comment out the `com.docker.ucp.mesh.http` label, and uncomment the `com.docker.lb.hosts`, `com.docker.lb.network`, and `com.docker.lb.port` labels (the new version for UCP 3.x).
7. Run another deployment using the opposite stack file and monitor the containers.  This time they will correctly deploy one-by-one.
