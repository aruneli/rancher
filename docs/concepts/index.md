---
title: Concepts
layout: default

---

## Concepts
---

In this section we introduce the key concepts in Rancher. You should be familiar with these concepts before attempting to use Rancher in production.

### Users

Rancher is a multi-user platform. Each user has a distinct view of the resources and containers. Rancher open source software is configured to authenticate against GitHub. In the future, Rancher will authenticate against multiple user directories such as LDAP or Active Directory. 

To learn more about how to add users, please read about [access control]({{site.baseurl}}/docs/configuration/access-control/), which needs to be configured to add users.

<a id="host"></a>

### Hosts

Hosts are the most basic unit of resources. A host represents a Linux server with a certain amount of CPU, memory, and local disk storage resources. A host must have network connectivity and must be able to reach the Rancher server. A host belonging to the same user must be able to reach each other. A host needs to have only one IP address (although it does not hurt to have multiple IPs associated with a host, like when hosts use public and private IP addresses for Internet-facing and internal traffic.)
<span class="highlight">Need more details on "host belonging to same user must be able to reach each other" or do we want to place in the add host section?</span>

Please read more about how to get started with [hosts]({{site.baseurl}}/docs/infrastructure/hosts).

### Environments

It is sometimes desirable to use different resource pools for different workloads. For example, you may want to separate dev/test workload from production workload. A Rancher environment is a resource pool.

Each user may have access to one or more environments. A user may create new environments. The owner of an environment can grant other users to access the environment.

[Access control]({{site.baseurl}}/docs/configuration/access-control/) will need to be set up before being able to [share environments]({{site.baseurl}}/docs/configuration/environments/) with users. 

### Networking

Rancher implements an overlay network for containers. Each container retains its usual IP address provided by the Docker daemon. Rancher provides the container with an additional IP address in the overlay network.

Containers in the same environment can communicate with each other using the Rancher-assigned IP address.

The Rancher-assigned IP address is not present in Docker meta-data and does not appear in the result of docker inspect. This sometimes causes incompatibilities with certain tools. We are working with the Docker community to make sure a future version of Docker can handle overlay network more cleanly.

### Service Discovery

Rancher adopts the standard Docker Compose terminology for services. A service is simply a group of containers created from the same Docker image.

There are two ways for containers implementing a service to be discovered: by registering itself behind a load balancer or by registering itself in the DNS service.

Rancher provides a managed load balancer service and a managed distributed DNS service. Load balancers have built-in health checks. Rancher additionally manages a health check services. Rancher removes unhealthy containers from the DNS service.

Rancher services can link to services implemented by other containers or to external services defined as a web services API.

### Load Balancer

Rancher implements a managed load balancer service using HAProxy. A load balancer service can scale to multiple hosts.

There are two ways to use load balancers. You can add individual containers to a load balancer manually. Alternatively, you can add a service to a load balancer. If you add a service to a load balancer, all containers implementing that service will be added to the load balancer automatically.

### Distributed DNS Service

Rancher implements a distributed DNS service using our own light-weight DNS server coupled with a distributed control plane. Each healthy container is automatically added to the DNS service. When queried by the service name, the DNS service returns a randomized list of IP addresses of the healthy containers implementing that service.

Because Rancher’s overlay networking provides each container with a distinct IP address, we do not need to deal with port mappings and do not need to handle situations like the same service listening on different ports. As a result, a simple DNS service is adequate for handling service discovery.

### Health Checks

Rancher implements a distributed health checker by running an HAProxy instance on every host. These HAProxy instances are used for the sole purpose of health checking and not used for load balancing. Each container is checked by up to three HAProxy instances running on different hosts. The container is considered healthy if it passes health check with at least one of the HAProxy instances.

Rancher’s approach handles network partitions and is more efficient than client-based health checks. By using HAProxy to perform health checks, Rancher enables users to specify the same health check policy for DNS service and for load balancers.

Depending on the result of health checks, a container is either in green or red state. A service is in green state if all the containers implementing that service are in green state. If all the containers are in red state the service is in red state. A service is in yellow state if Rancher has detected some of the containers in red state and is performing an action to remedy the situation.

### Service HA

Rancher can ensure that a fixed number of healthy containers are present in a service and restart new containers upon host crash or container failure.

### Service Upgrade

To upgrade a service in Rancher, the user clones that service and gives it a new name. The cloned service runs a new version of Docker image. The cloned service retains all the links of the original service but is not linked to by any of the existing services. The cloned service can therefore be tested in isolation. Once the cloned service passes the test, it can be put into production by mapping to the original service name or adding to a load balancer.

### Rancher Compose

Rancher implements a command-line tool called rancher-compose that is modeled after docker-compose. It takes in the same docker-compose.yml templates and deploys the application on Rancher. The rancher-compose tool additionally takes in a rancher-compose.yml file which extends and overwrites the docker-compose.yml file. The rancher-compose.yml file specifies attributes not present in standard docker-compose.yml files, such as the number of containers desired in a service, load balancing rules, and health check policies.

### Projects
Rancher project defines a scope of service discovery. It can be specified as an argument when running rancher-compose. For example:

```bash
rancher-compose up -p app1
```

This command deploys the docker-compose.yml template in the current directory into app1. All services in the same project can link to each other through service discovery.

### Scheduling

Rancher implements scheduling policies that are modeled closely after Docker Swarm. Just like Docker Swarm, Rancher implements host-based and label-based affinity and anti-affinity scheduling policies. You can run Docker Swarm on Rancher directly, but rancher-compose requires certain extensions in scheduling policies not present in Docker Swarm. Extension to Docker Swarm include the global scheduling policy, where Rancher ensures an instance of a particular service exists on every host. (Son to verify and add more details here.)

### Sidekicks

Rancher implements a special scheduling directive for the sidekick pattern. If service A is a sidekick to service B, they must be scheduled and scaled in lock step. A service can have multiple sidekicks. The volumes-from directive only works between sidekicks. Sidekicks is somewhat similar to Kubernetes pods although it is limited to scheduling and does not imply namespace sharing. (Alena to review and add more details)
