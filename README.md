> [!WARNING]  
> This software is currently in a beta release. For this reason, we don't recommend this for immediate usage while running production workloads. Any feedback is greatly appreciated!

# Equinix Load Balancers API Quickstart

[![Stability: Beta](https://img.shields.io/badge/stability-beta-33bbff.svg)]()

This document will describe how you can get started with Equinix Load Balancers via the API.

## Authentication

Before getting started, we'll gather the necessary components we'll need to be able to make our API calls. This will include the ID of the project that we wish to create loadbalancers in, an API token and then finally exchanging our API token for a JWT that will be used to make any requests.

### Get Project ID

#### Console URL
If you're not already familiar with the project ID that you want to use, you can get this from the URL in your console UI:

```bash
https://console.equinix.com/projects/<project-uuid>
```

#### Metal CLI

If you're already using the `metal` [CLI](https://github.com/equinix/metal-cli/tree/main), you can also get your project ID using the following:

```bash
╰─ metal project get
+--------------------------------------+---------------------------------+-------------------------------+
|                  ID                  |              NAME               |            CREATED            |
+--------------------------------------+---------------------------------+-------------------------------+
| < project id >                       | < project name >                |  < timestamp >                | 
+--------------------------------------+---------------------------------+-------------------------------+
```

Or if your metal cli is already configured, you can ensure the appropriate environment variables are set with: 

```bash
eval $(metal env --export)
```

This will put the default (from console) METAL_PROJECT_ID and METAL_AUTH_TOKEN in the environment.

### Retrieve Token

To make requests, you will need an API token. If you already have one, you can skip ahead to the next step. If you need instructions on how to create one, you can refer to the documentation [here](https://deploy.equinix.com/developers/docs/metal/accounts/api-keys/#creating-project-api-keys).

### Exchange Token

Up to this point, we've used constructs that should be familiar if you've used the Metal service previously. This next step is a new component of the service and involves exchanging our API token for a short lived JWT that we will use for auth in any load balancer request we want to make. To do this you can run the following

```bash
export METAL_AUTH_TOKEN=< your api token>
curl -X POST https://iam.metalctrl.io/api-keys/exchange -H "Authorization: bearer $METAL_AUTH_TOKEN"
```

The result of this will be an `access_token` which you can use in future calls in the format of:

```bash
curl -X POST -H "Authorization: bearer < access_token > ........
```

## Load Balancers

The first thing to do when working with loadbalancers is to create one! Once this is done, you'll have an IP address from your desired location which you will be able to route traffic through. To interact with them, you can run the below commands

### Create

Create a loadbalancer

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: bearer < access_token >" https://lb.metalctrl.io/v1/projects/< project id>/loadbalancers -d '{ "name": "test-loadbalancer", "location_id": "<location id >", "provider_id": "< provider id >"}'
```
#### Fields

|      Name       |  Type  | Required |                              Description                                 |
|-----------------|--------|----------|--------------------------------------------------------------------------|
| name            | String |    X     | Name of the loadbalancer                                                 |
| location_id     | ID     |    X     | ID of the location to deploy the loadbalancer. Current options are : <br>&nbsp;&nbsp;&nbsp;&nbsp;- **DA**: `lctnloc--uxs0GLeAELHKV8GxO_AI`<br>&nbsp;&nbsp;&nbsp;&nbsp;- **NY**: `lctnloc-Vy-1Qpw31mPi6RJQwVf9A`<br>&nbsp;&nbsp;&nbsp;&nbsp;- **SV**: `lctnloc-H5rl2M2VL5dcFmdxhbEKx`|
| port_ids        | [ID]   |    -     | Array of port ID's to be added to the loadbalancer                                                                                    |
| provider_id     | ID     |    X     | ID of the provider to deploy the loadbalancer. Currently only `loadpvd-gOB_-byp5ebFo7A3LHv2B` is supported                            |

#### Return

| Name   |                  Description                      |
|--------|---------------------------------------------------|
| id     | id of created loadbalancer                        |
| errors | error indicating why loadbalancer was not created |

### Update

Update fields of a specific loadbalancer

```bash
curl -X PATCH -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/< loadbalancer id >
```

#### Fields

|      Name       |       Type      |                              Description                                 |
|-----------------|-----------------|--------------------------------------------------------------------------|
| name            | String          | Name of the loadbalancer                                                 |
| add_port_ids    | [ID]            | Array of port ID's to be added to loadbalancer                           |
| remove_port_ids | [ID]            | Array of port ID's to be removed from loadbalancer                       |
| clear_ports     | Boolean         | Boolean indicating whether all ports should be removed from loadbalancer |

#### Return

| Name   |                  Description                      |
|--------|---------------------------------------------------|
| id     | id of updated loadbalancer                        |
| errors | error indicating why loadbalancer was not updated |

### Delete

Delete a specific loadbalancer

```bash
curl -X DELETE -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/< loadbalancer id >
```

#### Return

| Name   |                  Description                      |
|--------|---------------------------------------------------|
| id     | id of deleted loadbalancer                        |
| errors | error indicating why loadbalancer was not deleted |

### Get Individual Loadbalancer

Get a specific loadbalancer

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token>" https://lb.metalctrl.io/v1/loadbalancers/< loadbalancer id>
```

#### Return

|      Name                             |                          Description                             |
|---------------------------------------|------------------------------------------------------------------|
| data.loadBalancer.id                  | ID of the requested loadbalancer                                            
| data.loadBalancer.ports               | Array describing ports associated with requested loadbalancer
| data.loadBalancer.ports.id            | ID of the loadbalancer port
| data.loadBalancer.ports.name          | Name of the loadbalancer port
| data.loadBalancer.ports.number        | Number of the loadbalancer port
| data.loadBalancer.pools               | Array describing pools associated with requested loadbalancer
| data.loadBalancer.pools.id            | ID of the loadbalancer pool
| data.loadBalancer.pools.name          | Name of the loadbalancer pool
| data.loadBalancer.pools.created_at    | Timestamp of loadbalancer pool creation
| data.loadBalancer.pools.updated_at    | Timestamp of loadbalancer pool update
| data.loadBalancer.location.id         | ID of the location of the requested loadbalancer
| data.loadBalancer.location.name       | Name of the location of the requested loadbalancer
| data.loadBalancer.location.created_at | Timestamp of location creation
| data.loadBalancer.location.updated_at | Timestamp of location update
| data.loadBalancer.IPAddresses         | Array of IPAddresses associated with requested loadbalancer
| data.loadBalancer.IPAddresses.ip      | IP address associated with requested loadbalancer
| data.loadBalancer.provider            | ID of the provider associated with requested loadbalancer
| data.loadBalancer.created_at          | Timestamp of requested loadbalancer creation
| data.loadBalancer.updated_at          | Timestamp of requested loadbalancer update

### Get Project Loadbalancers

Get all loadbalancers associated with a given project

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token>" https://lb.metalctrl.io/v1/projects/<project id>/loadbalancers
```

#### Return 

|      Name                             |                          Description                             |
|---------------------------------------|------------------------------------------------------------------|
| loadbalancers                         | Array of loadbalancers belonging to a project
| loadbalancers.id                      | ID of the loadbalancer                                         
| loadbalancers.name                    | Name of the loadbalancer
| loadbalancers.IPAddresses             | Array of IPAddresses associated with loadbalancer
| loadbalancers.IPAddresses.ip          | IP address associated with loadbalancer
| loadbalancers.ports                   | Array describing ports associated with the loadbalancer
| loadbalancers.ports.id                | ID of the loadbalancer port
| loadbalancers.ports.name              | Name of the loadbalancer port
| loadbalancers.ports.number            | Number of the loadbalancer port
| loadbalancers.pools                   | Array describing pools associated with the loadbalancer
| loadbalancers.pools.id                | ID of the loadbalancer pool
| loadbalancers.pools.name              | Name of the loadbalancer pool
| loadbalancers.pools.created_at        | Timestamp of loadbalancer pool creation
| loadbalancers.pools.updated_at        | Timestamp of loadbalancer pool update
| loadbalancers.location.id             | ID of the location of the loadbalancer
| loadbalancers.location.name           | Name of the location of the loadbalancer
| loadbalancers.location.created_at     | Timestamp of location creation
| loadbalancers.location.updated_at     | Timestamp of location update
| loadbalancers.created_at              | Timestamp of requested loadbalancer creation
| loadbalancers.updated_at              | Timestamp of requested loadbalancer update

## Load Balancer Ports

### Create

Create loadbalancer port

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: bearer < access_token >" https://lb.metalctrl.io/v1/loadbalancers/< loadbalancer id>/ports -d '{ "name": "my-port", "number": 8910}'
```

#### Fields

|      Name       |       Type |   Required   |                              Description                  |
|-----------------|------------|--------------|-----------------------------------------------------------|
| name            | String     |      X       | Name of the loadbalancer port                             |
| number          | Int.       |      X       | Number of the loadbalancer port.                          |
| pool_ids        | [ID]       |      -       |Array of pool ID's to attach the loadbalancer port to      |

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of created loadbalancer port                        |
| errors | error indicating why loadbalancer port was not created |

### Update

Update loadbalancer port

```bash
curl -X PATCH -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/ports/< loadbalancer id >
```

#### Fields

|      Name       |       Type |                             Description                   |
|-----------------|------------|-----------------------------------------------------------|
| name            | String     | Name of the loadbalancer port                             |
| number          | Int.       | Number of the loadbalancer port.                          |
| pool_ids.       | [ID]       | Array of pool ID's to attach the loadbalancer port to     |

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of updated loadbalancer port                        |
| errors | error indicating why loadbalancer port was not updated |

### Delete

Delete loadbalancer port

```bash
curl -X DELETE -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/ports/< loadbalancer port id >
```

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of deleted loadbalancer port                        |
| errors | error indicating why loadbalancer port was not deleted |

### Get All Ports For Specific Loadbalancer

Get loadbalancer port

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/< loadbalancer id>/ports/
```

#### Return 

|      Name                             |                          Description                             |
|---------------------------------------|------------------------------------------------------------------|
| ports                                 | Array of all ports belonging to a loadbalancer                   |
| ports.id                              | ID of port                                                       |
| ports.name                            | Name of port                                                     |
| ports.number                          | Port Number                                                      |
| ports.pool_ids                        | Array of IDs of pools that a port is assigned to                 |
| ports.created_at                      | Timestamp for port creation                                      |
| ports.updated_at                      | Timestamp for port update                                        |

### Get Specific Loadbalancer Port

Get a port for a loadbalancer by port number

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/< load balancer id>/ports/< port number >
```

#### Return

{"created_at":"2023-09-07T19:34:15.961869Z","id":"loadprt-JOmS8o-xwtjq3rlwZcKkU","name":"8000","number":8000,"updated_at":"2023-09-07T19:34:15.961869Z"}

| Name       |                  Description                           |
|------------|--------------------------------------------------------|
| id         | ID of specified loadbalancer port                      |
| name       | Name of port                                           |
| number     | Port number                                            |
| created_at | Timestamp of port creation                             |
| updated_at | Timestamp of last time port was updated                |


## Load Balancer Pools

### Create

```bash
curl -X POST -H'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/projects/< project id >/loadbalancers/pools -d '{ "name": "test-pool", "protocol": "tcp" }'
```

#### Fields

|      Name       |       Type |   Required   |                              Description                  |
|-----------------|------------|--------------|-----------------------------------------------------------|
| name            | String     |      X       | Name of the loadbalancer pool                             |
| protocol        | String     |      X       | Pool Protocol. Valid options are tcp or udp               |
| port_ids        | [ID]       |      -       | Array of port ID's to attach to the loadbalancer pool     |
| origin_ids      | [ID]       |      -       | Array of origin ID's to attach to the loadbalancer pool   |

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of created loadbalancer pool                        |
| errors | error indicating why loadbalancer pool was not created |

### Update

```bash
curl -X PATCH -H'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/< loadbalancer pool id>
```

#### Fields

|      Name       |       Type |                             Description                   |
|-----------------|------------|-----------------------------------------------------------|
| name            | String     |  Name of the loadbalancer pool                            |
| protocol        | String     |  Pool Protocol. Valid options are tcp or udp              |
| port_ids        | [ID]       |  Array of port ID's to attach to the loadbalancer pool    |
| origin_ids      | [ID]       |  Array of origin ID's to attach to the loadbalancer pool  |

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of updated loadbalancer pool                        |
| errors | error indicating why loadbalancer pool was not updated |

### Delete

```bash
curl -X DELETE -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/< loadbalancer pool id>
```

#### Return

| Name   |                  Description                           |
|--------|--------------------------------------------------------|
| id     | id of deleted loadbalancer pool                        |
| errors | error indicating why loadbalancer pool was not deleted |

### Get Individual Loadbalancer Pool

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/< loadbalancer pool id>
```

#### Return

|      Name                                   |                          Description                             |
|---------------------------------------------|------------------------------------------------------------------|
| data.loadBalancerPool.id                    | ID of loadbalancer pool                                          |
| data.loadBalancerPool.name                  | Name of loadbalancer pool                                        |
| data.loadBalancerPool.protocol              | Protocol of loadbalancer pool (tcp/udp)                          |
| data.loadBalancerPool.owner_id              | ID of owner of loadbalancer pool.                                |
| data.loadBalancerPool.origins               | Array of origins associated with loadbalancer pool               |
| data.loadBalancerPool.origins.id            | ID of associated origin.                                         |
| data.loadBalancerPool.origins.name.         | Name of associated origin                                        |
| data.loadBalancerPool.origins.target        | IP address of origin target                                      |
| data.loadBalancerPool.origins.port_number   | Port number of origin target                                     |
| data.loadBalancerPool.origins.active        | Boolean indicating whether origin is active or not               |
| data.loadBalancerPool.origins.created_at    | Timestamp of origin creation                                     |
| data.loadBalancerPool.origins.updated_at    | Timestamp of origin update                                       |
| data.loadBalancerPool.ports                 | Array of ports associated with loadbalancer pool                 |
| data.loadBalancerPool.ports.id              | ID of associated port                                            |
| data.loadBalancerPool.ports.name            | Name of associated port                                          |
| data.loadBalancerPool.ports.number          | Port number of associated port                                   |
| data.loadBalancerPool.ports.created_at      | Timestamp of port creation                                       |
| data.loadBalancerPool.ports.updated_at      | Timestamp of port update                                         |
| data.loadBalancerPool.ports.loadBalancer_id | ID of loadbalancer associated with port                          |
| data.loadBalancerPool.ports.pool_ids        | Array of pool IDs associated with port                           |
| data.loadBalancerPool.created_at            | Timestamp of pool creation                                       |
| data.loadBalancerPool.updated_at            | Timestamp of pool update                                         |

### Get All Loadbalancer Pools For Project

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/projects/< project id >/loadbalancers/pools
```

#### Return

|      Name                                   |                          Description                             |
|---------------------------------------------|------------------------------------------------------------------|
| pools                                       | Array of pools belonging to the project                          |
| pools.id                                    | ID of pool belonging to the project                              |
| pools.name                                  | Name of pool belonging to the project                            |
| pools.protocol                              | Protocol of pool belong to the project (tcp/udp)                 |
| pools.ports                                 | Array of ports belonging to the associated pool                  |
| pools.ports.id                              | ID of port belonging to the associated pool                      |
| pools.ports.number                          | Port number of port belonging to the associated pool             |
| pools.ports.name                            | Name of port belonging to the associated pool                    |
| pools.ports.created_at                      | Timestamp of port creation                                       |
| pools.ports.updated_at                      | Timestamp of port update                                         |
| pools.ports.loadbalancer_id                 | ID of loadbalancer that port is attached to                      |
| pools.ports.pool_ids                        | Array of pool IDs that port is associated with                   |
| pools.ports.origins                         | Array of origins associated to pool                              |
| pools.ports.origins.id                      | ID of associated origin                                          |
| pools.ports.origins.name                    | Name of associated origin                                        |
| pools.ports.origins.target                  | IP address of origin target                                      |
| pools.ports.origins.port_number             | Port number of origin target                                     |
| pools.ports.origins.active                  | Boolean indicating whether origin is active                      |
| pools.ports.origins.created_at              | Timestamp of origin creation                                     |
| pools.ports.origins.updated_at              | Timestamp of origin update                                       |
| pools.created_at                            | Timestamp of pool creation                                       |
| pools.updated_at                            | Timestamp of pool update                                         |

## Load Balancer Origins

### Create

Create a loadbalancer origin

```bash
curl -X POST -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/<loadbalancer pool id >/origins -d '{ "name": "test-origin", "target": "192.168.10.2", "port_number": 8080 }'
```

#### Fields

|      Name       |       Type |   Required   |                              Description                                                         |
|-----------------|------------|--------------|--------------------------------------------------------------------------------------------------|
| name            | String     |      X       | Name of the loadbalancer origin                                                                  |
| target          | String     |      X       | IP Address of target. Currenly only public IPV4 endpoints hosted on Equinix metal are supported. |
| port_number     | Int        |      X       | Port number for loadbalancer origin target                                                       |
| active          | Boolean    |      -       | Boolean indicating whether origin is active                                                      |

#### Return

| Name   |                  Description                             |
|--------|----------------------------------------------------------|
| id     | id of created loadbalancer origin                        |
| errors | error indicating why loadbalancer origin was not created |

### Update

Update existing loadbalancer origin

```bash
curl -X PATCH -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/origins/< loadbalancer origin id >
```

#### Fields

|      Name       |       Type |                             Description                                  |
|-----------------|------------|--------------------------------------------------------------------------|
| name            | String     | Name of the loadbalancer origin                                          |
| target          | String     | IP Address of target                                                     |
| port_number     | Int        | Port number for loadbalancer origin target                               |
| active          | Boolean    | Boolean indicating whether origin is active                              |

#### Return

| Name   |                  Description                             |
|--------|----------------------------------------------------------|
| id     | id of updated loadbalancer origin                        |
| errors | error indicating why loadbalancer origin was not updated |

### Delete

Delete a loadbalancer origin

```bash
curl -X DELETE -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/origins/< loadbalancer origin id >
```

#### Return

| Name   |                  Description                             |
|--------|----------------------------------------------------------|
| id     | id of deleted loadbalancer origin                        |
| errors | error indicating why loadbalancer origin was not deleted |


### Get All Origins For Pool

Get all loadbalancer origins belonging to a pool

```bash
curl -H 'Content-Type: application/json' -H "Authorization: bearer <access_token >" https://lb.metalctrl.io/v1/loadbalancers/pools/< loadbalancer pool id >/origins
```

#### Return

| Name                |                  Description                             |
|---------------------|----------------------------------------------------------|
| origins             | Array of origins associated with specified pool          |
| origins.id          | ID of associated origin                                  |
| origins.target      | IP Address of associated origin target                   |
| origins.port_number | Port number of associated origin target                  |
| origins.active      | Boolean indicated whether target is active               |
| origins.created_at  | Timestamp of origin creation                             | 
| origins.updated_at  | Timestamp of origin update                               |