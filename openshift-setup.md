# FAPI Self-Certification Environment Setup to OpenShift


## Intro

You should first login to openshift

Have the `kc-sig-fapi` images built using the *podman-compose*.

```shell
cd ~/Workspace/kc-sig-fapi

./generate-all.sh kcsigfapi www.certification.openid.net keycloak-fapi.apps.cluster-c767.dynamic.opentlc.com resource-server-fapi.apps.cluster-c767.dynamic.opentlc.com http://client-jwks-server-fapi.apps.cluster-c767.dynamic.opentlc.com/ aspsp

podman-compose up --build
```

## Push local image built to openshift registry

```shell
oc create -f ./ocp-config/image-registry-rt.yml -n openshift-image-registry

podman login -u $(oc whoami) -p $(oc whoami -t) image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com

podman push localhost/kc-sig-fapi_resource_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_resource_server

podman push localhost/kc-sig-fapi_client_jwks_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_client_jwks_server

podman push localhost/kc-sig-fapi_api_gateway_nginx image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_api_gateway_nginx

podman push localhost/kc-sig-fapi_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_server

# podman push localhost/kc-sig-fapi_keycloak image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_keycloak

oc delete -f ./ocp-config/image-registry-rt.yaml -n openshift-image-registry
```

## Deploy

```shell

oc new-project fapi

oc create secret tls sso-x509-https-secret --cert=./https/server.pem --key=./https/server-key.pem -n fapi

oc create configmap fapi-client-ca-config --from-file=client-ca.crt=./https/client-ca.pem -n fapi

./create-keycloak-scripts-config.sh

## add keycloak operator setup here

oc create -f ./ocp-config/keycloak-instance.yaml -n fapi 

oc create -f ./ocp-config/resource-server-d.yaml -n fapi 
oc create -f ./ocp-config/resource-server-svc.yaml -n fapi 
oc create -f ./ocp-config/resource-server-rt.yaml -n fapi 

oc create -f ./ocp-config/client-jwks-server-d.yaml -n fapi 
oc create -f ./ocp-config/client-jwks-server-svc.yaml -n fapi 
oc create -f ./ocp-config/client-jwks-server-rt.yaml -n fapi 

```

## Destroy the environment

```shell
oc delete project fapi                                 
oc delete is/kc-sig-fapi_api_gateway_nginx -n openshift
oc delete is/kc-sig-fapi_client_jwks_server -n openshift                                  
oc delete is/kc-sig-fapi_resource_server -n openshift
oc delete is/kc-sig-fapi_server -n openshift
podman rmi kc-sig-fapi_test_runner kc-sig-fapi_load_balancer kc-sig-fapi_httpd kc-sig-fapi_server kc-sig-fapi_api_gateway_nginx kc-sig-fapi_client_jwks_server kc-sig-fapi_resource_server
```