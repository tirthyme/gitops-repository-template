# GitOps Repository Template

This is an example of what the AppStudio GitOps repository might look like for a simple application.

In particular:
- New environments should be added and removed from `environments/overlays/(environment name)`
    - Environments may can environment-specific K8s resources (Deplyoments/ConfigMaps/etc); they should be defined in the overlay
- There is only one application. It is defined under `application/`
- Environment-specific application configuration should be defined in `application/overlays/(environment name)`
    - For example, environment variables that are set only in production, or only in development.
    - Resources (ConfigMaps, etc) that are defined only in staging, or only in production.
- Otherwise this repository uses the standard kustomize `base`/`overlays` pattern, where `base` contain resource defitions, and `overlays` contain patches against those resource definitions.

**Q**: Do we need (micro)services as well? As in, an Application is composed of 1 or more services?

## Resources

There are two types of resources defined in this repository:
- **Application**: An application contains one or more Kubernetes resources (Deployments/ConfigMaps/etc), representing an application to deploy.
    - Application resources may be customized (by kustomize) based on the environment they are targetting, for example, different environment variables in staging/production containing different database service credentials.

- **Environment**: An environment contains zero or more Kubernetes resources (Deployments/ConfigMaps/etc) that should be present within a particular OCP/K8s environment. 
    - Applications are deployed within environments.


## Tree Diagram

Here is the hierarchy of resources within the repository:

```
.
├── environments
│   ├── base
│   │   ├── cm-env-config-map.yaml
│   │   └── kustomization.yaml
│   └── overlays
│       ├── dev
│       │   └── kustomization.yaml
│       └── staging
│           └── kustomization.yaml
├── README.md
└── application
    ├── base
    │   ├── deployment-sample-workload.yaml
    │   ├── kustomization.yaml
    │   ├── route-sample-workload.yaml
    │   └── service-sample-workload.yaml
    └── overlays
        ├── dev
        │   └── kustomization.yaml
        └── staging
            └── kustomization.yaml
```



## Try it out

#### Prerequisites:
- Ensure you have `kustomize` installed, and that `kubectl` is pointing to a valid *OpenShift* cluster.

You can try it out by running the following commands:
```bash
# Dev Environment -------------------------------------------------------------

# Output the K8s resources that will be applied for the application 
kustomize build application/overlays/dev

# Apply the resources (then wait a moment)
kustomize build application/overlays/dev | kubectl apply -f -

# Retrieve the route of the application
kubectl get routes

# You should now be able to access the Route, and see the environments variables output by that Route:
# example:
# https://sample-workload-my-namespace.apps.(cluster hostname).openshift.com/env
# should contain:
# ANOTHER_ENV_VAR: another-value
# ENV_VAR_FROM_CONFIG_MAP: dev
# RESOURCE_ENVIRONMENT: dev
# (...)

# Notice that RESOURCE_ENVIRONMENT and ENV_VAR_FROM_CONFIG_MAP match the environment name, 'dev'.



# Staging Environment ---------------------------------------------------------

# Output the K8s resources that will be applied for the application 
kustomize build application/overlays/staging

# Apply the resources (then wait a moment)
kustomize build application/overlays/staging | kubectl apply -f -

# Retrieve the route of the application
kubectl get routes

# You should now be able to access the Route, and see the environments variables output by that Route:
# example:
# https://sample-workload-my-namespace.apps.(cluster hostname).openshift.com/env
# should contain:
# ANOTHER_ENV_VAR: another-value
# ENV_VAR_FROM_CONFIG_MAP: staging
# RESOURCE_ENVIRONMENT: staging
# (...)

# Notice that RESOURCE_ENVIRONMENT and ENV_VAR_FROM_CONFIG_MAP match the environment name, 'staging'.
```
