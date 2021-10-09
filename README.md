# Default setup

As a new Konnect account owner, your organization is setup by default as follows:

1. A single default ServiceHub Catalogue.
2. A single default Runtime Manager Runtime Group. 
3. A set of default Teams described by the table in the [Default Teams](#default-teams) section. 

In order to customize your accounts configuration for use across your organizational boundaries, you can modify this default setup by adding new ServiceHub catalogues as well as Runtime Manager Runtime Groups, and associating them with custom Konnect Teams. The [Enterprise customization example](#enterprise-customization-example) of this document provides an example end-to-end setup steps to demonstrate how you can customize your organization as such.    

## Default Teams

| Konnect Default Team | Description |  
| -------------------- | ----------- | 
| Organization Admin | Users have full access to all objects in the account. |  
| Service Admin | Users have full access to the default ServiceHub service catalogue. |  
| Runtime Admin | Users have full access to the default Runtime Manager runtime group. |

The access level of each of these teams is as follows:

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: organization-admin
spec:
  users:
  permissions:
    # Retrieve any service
    - krn:reg/us:org/acme-bank:catalog/*:service/*!retrieve
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/*:service/*!update
    # Remove service from catalog
    - krn:reg/us:org/acme-bank:catalog/*:service/*!delete
    # Create a new service under a catalog
    - krn:reg/us:org/acme-bank:catalog/*:service!create

    # Retrieve any catalog
    - krn:reg/us:org/acme-bank:catalog/*!retrieve
    # Update any catalog
    - krn:reg/us:org/acme-bank:catalog/*!update
    # Remove catalog
    - krn:reg/us:org/acme-bank:catalog/*!delete
    # Create a new catalog
    - krn:reg/us:org/acme-bank:catalog!create
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: service-admin
spec:
  users:
  permissions:
    # Retrieve any service from default catalog
    - krn:reg/us:org/acme-bank:catalog/default-catalog:service/*!retrieve
    # Update any service in default catalog
    - krn:reg/us:org/acme-bank:catalog/default-catalog:service/*!update
    # Remove service from default catalog
    - krn:reg/us:org/acme-bank:catalog/default-catalog:service/*!delete
    # Add a new service to default catalog
    - krn:reg/us:org/acme-bank:catalog/default-catalog:services!create
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: runtime-admin
spec:
  users:
  permissions:
    # Update an existing runtime
    - krn:reg/us:org/acme-bank:runtime-group/default-rg:runtime/*!update
    # Remove runtime from runtime group
    - krn:reg/us:org/acme-bank:runtime-group/default-rg:runtime/*!delete
    # Add runtime to runtime group
    - krn:reg/us:org/acme-bank:runtime-group/default-rg:runtime!create
```

####  Api Definition
<details>
  <summary>Option 1 - Flat team </summary>
	
```
openapi: 3.0.0
info:
  title: sample API for https://github.com/rezaloo75/konnect usecase
  description: this contains a subset of Konnect API needed to support the usecases described at https://github.com/rezaloo75/konnect 
  version: 0.0.1
paths:
  /teams:
    get:
      summary: get all teams
      responses:
        '200':
          description: paginated list of teams
    post:
      summary: craete a team
      description: this API allows to create a new team
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Team'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
  /teams/{id}:
    get:
      summary: get a team
      parameters:
        - name: id
          in: path
          description: team id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: a team
          content:
            application/json:
              schema:
               $ref: '#/components/schemas/Team'
      
    patch:
      summary: update a team
      description: this API allows to update a team
      parameters:
        - name: id
          in: path
          description: team id
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Team'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
    delete:
      summary: delete a team
      parameters:
        - name: id
          in: path
          description: team id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: confirmation that the teamw was delited.

components:
  schemas:
    KRN:
      type: string
      format: KRN
      description: Kong resource notations URI
    Permission:
      type: object
      description: "TODO: wildcard support still need to be defined"
      properties:
        krn:
           $ref: '#/components/schemas/KRN'
        action:
          type: string
          enum: [create, update, delete, etc]
        
    Team:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: faf67ef4-7d2d-474d-9f1c-08e996cbcf4c
        name:
          type: string
          example: Administrator
        description:
          type: string
          example: This is the team that allows you to do anything 
        created_at: 
          type: string
          format: date-time
          example: 2017-07-21T17:32:28Z
        updated_at:
          type: string
          format: date-time
          example: 2020-07-21T17:32:28Z
        permissions:
          type: array
          items:
            $ref: '#/components/schemas/Permission' 
        users:
          type: array
          items:
            $ref: '#/components/schemas/KRN'
      # Both properties are required
      required:  
        - id
        - name	
```	
</details>

<details>
  <summary>Option 2 - Users / Groups / Roles/ Permission </summary>

```
openapi: 3.0.0
info:
  title: sample API for https://github.com/rezaloo75/konnect usecase
  description: this contains a subset of Konnect API needed to support the usecases described at https://github.com/rezaloo75/konnect 
  version: 0.0.1
paths:
  /groups:
    get:
      summary: get all user groups
      responses:
        '200':
          description: paginated list of groups
    post:
      summary: craete a group
      description: this API allows to create a new group
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Group'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
  /group/{id}:
    get:
      summary: get a group
      parameters:
        - name: id
          in: path
          description: group id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: a group
          content:
            application/json:
              schema:
               $ref: '#/components/schemas/Group'
      
    patch:
      summary: update a group
      description: this API allows to update a group
      parameters:
        - name: id
          in: path
          description: group id
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Group'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
    delete:
      summary: delete a group
      parameters:
        - name: id
          in: path
          description: group id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: confirmation that the group was delited.
          
  /roles:
    get:
      summary: get all roles
      responses:
        '200':
          description: paginated list of roles
    post:
      summary: craete a role
      description: this API allows to create a new role
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Role'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
  /role/{id}:
    get:
      summary: get a role
      parameters:
        - name: id
          in: path
          description: role id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: a role
          content:
            application/json:
              schema:
               $ref: '#/components/schemas/Role'
      
    patch:
      summary: update a role
      description: this API allows to update a role
      parameters:
        - name: id
          in: path
          description: role id
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Role'
      responses: 
        '200':
          description: a message confirming the successful operation
          content:
            application/json:
              schema:
                type: object
    delete:
      summary: delete a role
      parameters:
        - name: id
          in: path
          description: role id
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: confirmation that the role was delited.


components:
  schemas:
    KRN:
      type: string
      format: KRN
      description: Kong resource notations URI
    Permission:
      type: object
      description: "TODO: wildcard support still need to be defined"
      properties:
        krn:
           $ref: '#/components/schemas/KRN'
        action:
          type: string
          enum: [create, update, delete, etc]
    Group:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: faf67ef4-7d2d-474d-9f1c-08e996cbcf4c
        name:
          type: string
          example: AdministratorGroup
        description:
          type: string
          example: This is the group that all administrator belongs to
        created_at: 
          type: string
          format: date-time
          example: 2017-07-21T17:32:28Z
        updated_at:
          type: string
          format: date-time
          example: 2020-07-21T17:32:28Z
        roles:
          type: array
          items:
            $ref: '#/components/schemas/KRN' 
        users:
          type: array
          items:
            $ref: '#/components/schemas/KRN'
    Role:
      type: object
      properties:
        id:
          type: string
          format: uuid
          example: faf67ef4-7d2d-474d-9f1c-08e996cbcf4c
        name:
          type: string
          example: AdministratorRole
        description:
          type: string
          example: This is the Role that describe all Permission  
        created_at: 
          type: string
          format: date-time
          example: 2017-07-21T17:32:28Z
        updated_at:
          type: string
          format: date-time
          example: 2020-07-21T17:32:28Z
        permissions:
          type: array
          items:
            $ref: '#/components/schemas/Permission' 
      # Both properties are required
      required:  
        - id
        - name
```
	
</details>

***

In the next section, we will see how this default starting point state of a Konnect organization can be modified to allow for more complex Enterprise organization setups through an example. 

***_Open Question_***: _This setup does not allow for the more fine grained Service Developer and Service Page Editor roles we have today in Konnect. We need to conclude on how to address that, if at all needed._  

# Enterprise customization example

As a pre-condition to the steps outlined in this section we will assume that your Konnect account has been configured for OIDC SSO federation with an IdP as described in [this documentation section](http://google.com).

As a new Konnect account owner, you need to start by setting up your account for use by your end-users. This document describes the steps you need to take to do so. To illustrate these steps, we will assume a sample scenario that has the following organizational setup. 

## Scenario

ACME Bank is our sample organization. ACME bank consists of the the teams listed in the table below. Each of these teams interacts with their Konnect organization as per the description in the following table. 

| Team | Description |
| ---- | ----------- |
| Retail | The members of the Retail Konnect team are actively building Banking applications designed for the Retail business unit of the Bank that is concerned with the Bank's customer's day-to-day banking needs. These are applications such as the Bank's mobile application for consumers. |
| Investment | The members of the Investment Konnect team are actively building Investment applications designed for providing trading services to the Bank's consumers. |
| Operations | The members of the Operations team are responsible for deploying the services built by the Retail and Investment teams to the Bank's official Production environment. From a Konnect perspective, deployment to Production environment means that the Service is proxied on the appropriate set of Gateway data-planes (which are provisioned and managed by the Operations team) and available for usage by its clients. 

From a Konnect Services perspective, there are three scopes of services (from a scope of control perspective for the teams listed) as described by the table below.

| Service Scope | Scope of Control | 
| ------------ | ---------------- | 
| Retail Specific | Only the Retail team should be able to view, consume and manage the lifecycle of these services. The members of the Investment team should not need to see these services at all. |
| Investment Specific | Only the Investment team should be able to see and manage the lifecycle of these services. The members of the Investment team should not need to see these services at all. |
| Common | These are services that can be created by any team at ACME, and that are available for all teams to view and consume. However, only the users in the teams that own the service have lifecycle management responsibilities for them. |

The goal of the steps outlined in this document is to enable a user with account ownership role (that is full root access to the Konnect account) to be able to setup the organization so that ACME Banks users can manage the lifecycle of their services according to their team membership and the types of services they are responsible for. 
 
## Setting up ServiceHub permissions

In this section, we will proceed to:

1. Create the Service Catalogues that allow for the three different scopes of services that exist at ACME.
2. Teams that reflect the permissions associated with the Retail, Investment, and all ACME bank users as users who log into Konnect and need to interact with ServiceHub. 
3. The role mappings that allow users to map to the appropriate teams as they log in.  

As an account owner, let's assume that you have an account that is as follows:

```
apiVersion: konnect.kong.io/v1
kind: Organization
metadata:
  name: acme-bank
```

We start by creating four teams as follows.

1 - Let's create three teams, each of which is designed to allow the permissions to the common, investment, and retail service catalogues respectively. 

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: view-common-services
spec:
  users:
  permissions:
    # List service in the catalog
    - krn:reg/us:org/acme-bank:catalog/*!list
     # Read common service in the catalog
    - krn:reg/us:org/acme-bank:catalog/common-service*!retrieve
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-retail-services
spec:
  users:
  permissions:
    # List service in the catalog
    - krn:reg/us:org/acme-bank:catalog/*!list
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/retail-service*!update
    # Remove service from catalog
    - krn:reg/us:org/acme-bank:catalog/retail-service*!delete
    # Add a new service to catalog
    - krn:reg/us:org/acme-bank:catalog/retail-service*!create
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-investment-services
spec:
  users:
  permissions:
    # List service in the catalog
    - krn:reg/us:org/acme-bank:catalog/*!list
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/investment-service*!update
    # Remove service from catalog
    - krn:reg/us:org/acme-bank:catalog/investment-service*!delete
    # Add a new service to catalog
    - krn:reg/us:org/acme-bank:catalog/investment-service*!create
```

3 - We then create the role mappings to allow each user logging in to map to the teams we create above appropriately based on the OIDC log-in claims.  

<details>
  <summary>Option 2 - Identity Only SSO </summary>
	
	If SSO is used for `only` Identity federation, it make no sense to map IdP user to teams.
	Team management will have to be handle within connect. Ence nothing else need to be done here.
	
</details>

<details>
  <summary>Option 2 - Role assumption </summary>
	
	If SSO is used for both Identity and Role federation, than we should build (Teams Option 2) and add support for "role assumption".
	
	Concretely this means that:
	
	- an OIDC/SAML integration will map to a Role or Roles
	
	- an OIDC/SAML integration will map conditionally to a Role/Roles based on a custom OIDC claim or SAML assertion
	
	- (if multiple roles are supported, the user will be prompted to pick the role that should be assumed for the session)
	
	API definition will look very similar to
	
	https://konnect.konghq.com/docs/#/idps%2F%3Aidp_id%2Fgroup_mappings/getMany
	
</details>



## Setting up Runtime Manager permissions

In this section, we are going to setup the Runtime Manager aspects of Konnect. The goal will be to allow each of the two application teams at ACME Bank, that is the Retail and Investment teams, to have their own Runtime Group (that is a group of Kong Gateways that they control) as a sandbox for deploying their Services, while having a common "Production" Runtime Group that is designated for the deployment of services from both the Retail and Investment teams.

__Note:__ Runtime Groups will be an Enterprise tier feature only and therefore their usage limited to our Enterprise user base. Of this user base, we expect that a single organization will have on average around 6-12 runtime groups and no more than 100 or so at max. 

1 - Let's begin by creating the three Runtime Groups. 

```sh
curl --request POST \
  --url https://konnect.konghq.com/runtimegroups \
  --header 'Content-Type: application/json' \
  --data '{
  "id": "1",
  "name": "retail-sandbox-rg",
  "labels": {
    "environment": "sandbox"
  },
  "config": {}
}'
```

```sh
curl --request POST \
  --url https://konnect.konghq.com/runtimegroups \
  --header 'Content-Type: application/json' \
  --data '{
  "id": "2",
  "name": "investment-sandbox-rg",
  "labels": {
    "environment": "sandbox"
  },
  "config": {}
}'
```

```sh
curl --request POST \
  --url https://konnect.konghq.com/runtimegroups \
  --header 'Content-Type: application/json' \
  --data '{
  "id": "3",
  "name": "acme-production-rg",
  "labels": {
    "environment": "production"
  },
  "config": {}
}'
```

2 - Let's now give access for each runtime group by creating the appropriate teams. Given an ***update*** permission on a runtime group, a user can:

- Change/refresh the provisioning key. 
- Change the runtime group name or description.  
- Add/delete/modify attributes to the runtime group. 
- Add/delete/modify (common) runtime group configuration. 

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-retail-sandbox
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:runtime-group/retail-sandbox-rg:/*!update
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-investment-sandbox
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:runtime-group/investment-sandbox-rg:/*!update
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-production
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:runtime-group/acme-production-rg:/*!update
```


## Summary of setup so far and final config to allow for production deployment

Based on the Service Catalogues, Runtime Groups, Teams, and IdP Mappings we have created so far, we can assert that the following statements stand true for the ACME Konnect account:

1. Users who are in the IdP Retail dev group of ACME Bank are able to create new services in the Retail service catalogue and these services will automatically be available to other members of their team. 
2. Users who are in the IdP Investment dev group of ACME Bank are able to create new services in the Investment service catalogue and these services will automatically be available to other members of their team.
3. Both Retail and Investment dev group members are able to publish their services to the ACME Common service catalogue so that they are available to any ACME Bank user who logs into ServiceHub and looks at the common catalogue.
4. All ACME Bank employees are able to view all services in the Konnect ServiceHub Common catalogue. 
5. Users who are in the IdP Retail dev group of ACME Bank are able to see and access the Retail Sandbox Runtime Group in Runtime Manager. They are thus able to:
    1. Start/stop Kong Gateway runtime instances that are associated with this runtime group. In practical terms this means these users may obtain a runtime group provisioning key for the Retail Sandbox Runtime Group to attach a new data plane to this group.
    2. Associate subsets of configuration from this runtime group with the services in the Service Catalogues that they have access to, which for them is the Retail and Common service catalogues.  
6. Users who are in the IdP Investment dev group of ACME Bank are able to see and access the Investment Sandbox Runtime Group in Runtime Manager. They are thus able to:
    1. Start/stop Kong Gateway runtime instances that are associated with this runtime group. In practical terms this means these users may obtain a runtime group provisioning key for the Investment Sandbox Runtime Group to attach a new data plane to this group.
    2. Associate subsets of configuration from this runtime group with the services in the Service Catalogues that they have access to, which for them is the Investment and Common service catalogues.   
7. Users who are in the ACME Operations IdP group are able to see and access the Production Runtime Group in Runtime Manager. They are thus able to: :
    1. Start/stop Kong Gateway runtime instances that are associated with this runtime group. In practical terms this means these users may obtain a runtime group provisioning key for the Production Runtime Group to attach a new data plane to this group.
    2. Associate subsets of configuration from this runtime group with the services in the Service Catalogues that they have access to, which for them is the Investment, Retailm and Common service catalogues.

Note that at this point, no Operations user has access to any Service Catalogue, hence, to allow for the operations teams to be able to associate configuration from the production runtime group to ACME's services, we need to setup the following permission to enable point 7.2 above:

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: deploy-all-services
spec:
  users:
  permissions:
    # Have access to any service version's configuration 
    - krn:reg/us:org/acme-bank:catalog/*:service/*!retrieve
```

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-acme-operations
spec:
  groupName: acme-operations
  teamName: deploy-all-services
```

With this step done, we have completed our configuration of ACME Bank's Konnect organization and all ACME Bank employees, be they from the Retail Team, Investment Team, Operations Team, or general employees not part of any of the above team, can log into Konnect an enjoy a complete service connectivity experience!

## Using Konnect with the setup in place

### As a member of the Retail development team 

Let's begin by seeing the catalogues that I have access to:

```
GET "https://konnect.konghq.com/api/catalogue

catalogues:
  common-catalogue:
  	ID: 1
  	name: Common
  	service-count: 0
  retail-catalogue:
  	ID: 2
  	name: Retail 
  	service-count: 0
  count: 2
```

Note that I don't see the finance catalogue, that is because I don't have access to it. Let's now create a an exchange rate service and publish it to the common catalogue:

```
POST "https://konnect.konghq.com/api/catalogue/1

name: Exchange Rate
description: Provides the current and historical global exchange rates
versions:
	version: v1.0
	version-description: Initial version of the Exchange Rate service
```

If we list out the services in the common catalogue we should now see our new Exchange service listed:

```
GET "https://konnect.konghq.com/api/catalogue/1/service/*

services:
	ID: 1
	name: Exchange Rate
```

```
GET "https://konnect.konghq.com/api/catalogue/1/service/1

ID: 1
name: Exchange Rate
description: Provides the current and historical global exchange rates
versions:
	ID: 1
	name: v1.0
	version-description: Initial version of the Exchange Rate service
	config: 
```

Let's now associate some Kong Gateway proxy configuration with the retail sandbox runtime group:

```
GET https://konnect.konghq.com/api/runtime-group

runtime-groups:
  manage-retail-sandbox:
  	ID: 1
  	name: Retail Sandbox
  	# We will assume that a Kong Gateway runtime instance has been started 
  	# and associated with this instance already
  	provisioning-id: TY23910203FG3
  	runtime-count: 1
  	service-versions: 0
```

```
POST https://konnect.konghq.com/api/runtime-group/1/configuration

config-type: kong
associated-service: 1
associated-service-catalogue: 1
config-content: 
        service:
          connect_timeout: 60000
          host: l9l76.mocklab.io
          port: 80
          protocol: http
          read_timeout: 60000
          retries: 5
          write_timeout: 60000
          routes:
          - name: account_id
            paths:
            - /id
            path_handling: v0
            preserve_host: false
            protocols:
            - http
            regex_priority: 0
            strip_path: true
            tags:
            - these
            - are
            - tags
            - kong
            https_redirect_status_code: 426
            request_buffering: true
            response_buffering: true
          plugins:
          - name: key-auth
            config:
              anonymous: null
              hide_credentials: false
              key_in_body: false
              key_in_header: true
              key_in_query: true
              key_names:
              - apikey
              run_on_preflight: true

```
