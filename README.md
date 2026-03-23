# FIWARE Connector Deployment Notes

This repository deploys the FIWARE Data Space Connector into a tenant vcluster.

For tenant-specific examples below, replace `gx-participant1` with `<tenant>`. The
public tenant base host is assumed to be:

`https://<tenant>.dil.collab-cloud.eu`

## Services That Should Be Externally Exposed

Based on the FIWARE Data Space Connector documentation, the current deployment
needs these externally reachable entrypoints:

### 1. Keycloak

- Kubernetes service: `fiware-dsc-keycloak`
- In-cluster port: `8080`
- Public purpose: user-facing identity and OIDC/OAuth flows
- Public URL in this deployment: `https://<tenant>.dil.collab-cloud.eu/auth`

This path is already assumed by the deployment values through
`keycloak.realm.frontendURL`.

### 2. Verifier

- Kubernetes service: `verifier`
- In-cluster port: `3000`
- Public purpose: wallet-facing SIOPv2 / OIDC4VP / VC verification flows

The FIWARE DSC documentation describes the verifier as the component that
requests credentials from the wallet and issues access tokens after successful
verification. Because wallets and external users must reach it, it should be
published through the host-cluster Envoy Gateway.

This repo does not currently hardcode a public verifier path. Keep the public
mapping stable once chosen. For example, expose it either:

- on a dedicated host, or
- on a stable shared-host path such as `https://<tenant>.dil.collab-cloud.eu/verifier`

### 3. APISIX Data Plane

- Kubernetes service: `fiware-dsc-apisix-data-plane`
- In-cluster ports: `80`, `443`
- Public purpose: product-facing Policy Enforcement Point (PEP) / API gateway

The FIWARE DSC documentation describes APISIX as the gateway in front of the
protected product APIs. It should therefore be externally reachable, but in this
deployment it should be exposed through the host-cluster Envoy Gateway, not as a
Kubernetes `LoadBalancer` service inside the tenant cluster.

Use Envoy Gateway routes to publish the API paths that should be reachable by
consumers on:

- `https://<tenant>.dil.collab-cloud.eu/...`

The exact externally published paths depend on the APIs you enable behind APISIX.
In this repo:

- `tm-forum-api` is disabled
- `contract-management` is disabled
- `did` is disabled
- `dataSpaceConfig` is disabled
- `apisix.catchAllRoute` is disabled

So APISIX should only be exposed for the explicit product/API routes that you
configure.

## Services That Should Stay Internal

These services are internal-only in the current deployment and should normally
not be exposed through Envoy Gateway:

- `authentication-mysql`
- `data-service-postgis`
- `postgresql`
- `fiware-dsc-etcd`
- `fiware-dsc-apisix-control-plane`
- `credentials-config-service`
- `odrl-pap`
- `trusted-issuers-list`
- `data-service-scorpio`

They support storage, policy, registry, or control-plane functions for the
connector and are not the intended public entrypoints.

## Current Tenant Assumptions

The current values file assumes:

- tenant host: `https://<tenant>.dil.collab-cloud.eu`
- Keycloak published at `/auth`
- participant DID pattern: `did:web:<tenant>.dil.collab-cloud.eu:identity`

## Source Basis

This summary is based on:

- the FIWARE Data Space Connector architecture and deployment documentation
- the services created by this deployment in namespace `fiware-dsc`
- the current repo values in
  `plaform-apps/fiware-dsc/values.yaml`
