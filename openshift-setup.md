first must login to openshift

kc-sig-fapi build

```shell
cd ~/Workspace/obbr-kc-sig-fapi

./generate-all.sh kcsigfapi www.certification.openid.net keycloak-fapi.apps.cluster-c767.dynamic.opentlc.com resource-server-fapi.apps.cluster-c767.dynamic.opentlc.com http://client-jwks-server-fapi.apps.cluster-c767.dynamic.opentlc.com/ aspsp

podman-compose up --build
```

pushing local image built to openshift registry

```shell
oc create -f ./ocp-config/image-registry-rt.yml -n openshift-image-registry

podman login -u $(oc whoami) -p $(oc whoami -t) image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com

podman push localhost/obbr-kc-sig-fapi_resource_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/obbr-kc-sig-fapi_resource_server

podman push localhost/obbr-kc-sig-fapi_client_jwks_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/obbr-kc-sig-fapi_client_jwks_server

podman push localhost/obbr-kc-sig-fapi_api_gateway_nginx image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/obbr-kc-sig-fapi_api_gateway_nginx

podman push localhost/obbr-kc-sig-fapi_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/obbr-kc-sig-fapi_server

# podman push localhost/obbr-kc-sig-fapi_keycloak image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/obbr-kc-sig-fapi_keycloak

oc delete -f ./ocp-config/image-registry-rt.yaml -n openshift-image-registry
```


creating deployments
```shell

oc new-project fapi

oc create secret tls sso-x509-https-secret --cert=./https/server.pem --key=./https/server-key.pem

oc create configmap fapi-client-ca-config --from-file=client-ca.crt=./https/client-ca.pem

oc create -f ./ocp-config/keycloak-instance.yaml -n fapi 

oc create -f ./ocp-config/resource-server-d.yaml -n fapi 
oc create -f ./ocp-config/resource-server-svc.yaml -n fapi 
oc create -f ./ocp-config/resource-server-rt.yaml -n fapi 

oc create -f ./ocp-config/client-jwks-server-d.yaml -n fapi 
oc create -f ./ocp-config/client-jwks-server-svc.yaml -n fapi 
oc create -f ./ocp-config/client-jwks-server-rt.yaml -n fapi 

```



destroy

```shell
oc delete project fapi                                 
oc delete is/obbr-kc-sig-fapi_api_gateway_nginx -n openshift
oc delete is/obbr-kc-sig-fapi_client_jwks_server -n openshift                                  
oc delete is/obbr-kc-sig-fapi_resource_server -n openshift
oc delete is/obbr-kc-sig-fapi_server -n openshift
podman rmi obbr-kc-sig-fapi_test_runner obbr-kc-sig-fapi_load_balancer obbr-kc-sig-fapi_httpd obbr-kc-sig-fapi_server obbr-kc-sig-fapi_api_gateway_nginx obbr-kc-sig-fapi_client_jwks_server obbr-kc-sig-fapi_resource_server
```