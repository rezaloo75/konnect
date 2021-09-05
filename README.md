# Enterprise Teams setup example

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

As a pre-condition to the steps outlined in this section we will assume that your Konnect account has been configured for OIDC SSO federation with an IdP as described in [this documentation section](http://google.com). In this section, we will proceed to:

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


1 - Let's begin by creating the three types of service catalogues, each associated with one of the service scopes we described in the previous section. 

```
apiVersion: konnect.kong.io/v1
kind: Catalog
metadata:
  name: common-catalog
spec:
```
```
apiVersion: konnect.kong.io/v1
kind: Catalog
metadata:
  name: retail-catalog
spec:
```
```
apiVersion: konnect.kong.io/v1
kind: Catalog
metadata:
  name: investment-catalog
spec:
```

2 - Now let's create three teams, each of which is designed to allow the permissions to the common, investment, and retail service catalogues respectively. 

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: view-common-services
spec:
  users:
  permissions:
    - krn:reg/us:org/acme-bank:catalog/common-catalog:service/*!view
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-retail-services
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/retail-catalog:service/*!update
    # Remove service from catalog
    - krn:reg/us:org/acme-bank:catalog/retail-catalog:service/*!delete
    # Add a new service to catalog
    - krn:reg/us:org/acme-bank:catalog/retail-catalog:services!create
```

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: manage-retail-services
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/investment-catalog:service/*!update
    # Remove service from catalog
    - krn:reg/us:org/acme-bank:catalog/investment-catalog:service/*!delete
    # Add a new service to catalog
    - krn:reg/us:org/acme-bank:catalog/investment-catalog:services!create
```

3 - We then create the role mappings to allow each user logging in to map to the teams we create above appropriately based on the OIDC log-in claims.  

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-acme-to-team-view-common-services
spec:
  groupName: acme
  teamName: view-common-services
```

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-retai-dev
spec:
  groupName: retail-dev
  teamName: manage-retail-services
```

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-investment-dev
spec:
  groupName: investment-dev
  teamName: manage-ivestment-services
```

## Setting up Runtime Manager permissions

In this section, we are going to setup the Runtime Manager aspects of Konnect. The goal will be to allow each of the two application teams at ACME Bank, that is the Retail and Investment teams, to have their own Runtime Group (that is a group of Kong Gateways that they control) as a sandbox for deploying their Services, while having a common "Production" Runtime Group that is designated for the deployment of services from both the Retail and Investment teams.

1 - Let's begin by creating the three Runtime Groups. 

```
apiVersion: konnect.kong.io/v1
kind: RuntimeGroup
metadata:
  name: retail-sandbox-rg
spec:
```

```
apiVersion: konnect.kong.io/v1
kind: RuntimeGroup
metadata:
  name: investment-sandbox-rg
spec:
```

```
apiVersion: konnect.kong.io/v1
kind: RuntimeGroup
metadata:
  name: acme-production-rg
spec:
```

2 - Let's now give access for each runtime group by creating the appropriate teams. 

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
    - krn:reg/us:org/acme-bank:runtime-group/retail-sandbox-rg:/*!delete
    - krn:reg/us:org/acme-bank:runtime-group/retail-sandbox-rg!create
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
    - krn:reg/us:org/acme-bank:runtime-group/investment-sandbox-rg:/*!delete
    - krn:reg/us:org/acme-bank:runtime-group/investment-sandbox-rg!create
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
    - krn:reg/us:org/acme-bank:runtime-group/acme-production-rg:/*!delete
    - krn:reg/us:org/acme-bank:runtime-group/acme-production-rg:!create
```

3 - Finally, we can now create new IDP mappings to make sure that the members of the operations team have access to the production runtime group, while the members of the retail and investment dev teams have access to their respective sandbox runtime groups. 

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-acme-operations
spec:
  groupName: acme-operations
  teamName: manage-production
```

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-retai-ops
spec:
  groupName: retail-dev
  teamName: manage-retail-sandbox
```

```
apiVersion: konnect.kong.io/v1
kind: IDPMapping
metadata:
  name: group-investment-ops
spec:
  groupName: investment-dev
  teamName: manage-investment-sandbox
```

## Wrap-up and final config to allow for production deployment

Based on the Service Catalogues, Runtime Groups, Teams, and IdP Mappings we have created so far, we can assert that the following statements stand true for the ACME Konnect account:

1. Users who are in the IdP Retail dev group of ACME Bank are able to create new services in the Retail service catalogue and these services will automatically be available to other members of their team. 

2. Users who are in the IdP Investment dev group of ACME Bank are able to create new services in the Investment service catalogue and these services will automatically be available to other members of their team.

3. Both Retail and Investment dev group members are able to publish their services to the ACME Common service catalogue so that they are available to any ACME Bank user who logs into ServiceHub and looks at the common catalogue.

4. All ACME Bank employees are able to view all services in the Konnect ServiceHub Common catalogue. 

4. Users who are in the IdP Retail dev group of ACME Bank are able to see and access the Retail Sandbox Runtime Group in Runtime Manager. They are thus able to:
	5.	Start/stop Kong Gateway runtime instances that are associated with this runtime group.
	6. Deploy their Services from ServiceHub to this runtime group 
5. Users who are in the IdP Investment dev group of ACME Bank are able to see and access the Investment Sandbox Runtime Group in Runtime Manager. They are thus able to:
	5.	Start/stop Kong Gateway runtime instances that are associated with this runtime group.
	6. Deploy their Services from ServiceHub to this runtime group 
6. Users who are in the ACME Operations IdP group are able to see and access the Production Runtime Group in Runtime Manager. They are thus able to: :
	5.	Start/stop Kong Gateway runtime instances that are associated with this runtime group.
	6. Deploy any Service to which they have Deploy access from ServiceHub to this runtime group

Note that at this point, no Operations user has access to any Service Catalogue, hence, to allow for the operations teams to be able to deploy the services to production, we need to setup the following permission to enable point 7.2 above:

```
apiVersion: konnect.kong.io/v1
kind: Team
metadata:
  name: deploy-all-services
spec:
  users:
  permissions:
    # Update any service
    - krn:reg/us:org/acme-bank:catalog/*:service/*!deploy
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
