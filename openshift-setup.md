first must login to openshift

setup

```shell
cd ~/Workspace/obbr-kc-sig-fapi

./generate-all.sh kcsigfapi www.certification.openid.net keycloak-fapi.apps.cluster-c767.dynamic.opentlc.com rs-endpoint-fapi.apps.cluster-c767.dynamic.opentlc.com http://client-jwks-server-fapi.apps.cluster-c767.dynamic.opentlc.com/ aspsp

oc new-project fapi

oc create secret tls sso-x509-https-secret --cert=./https/server.pem --key=./https/server-key.pem

oc create configmap fapi-client-ca-config --from-file=client-ca.crt=./https/client-ca.pem

docker-compose up --build
```

pushing local image built to openshift registry

```shell

oc create -f ./ocp-config/keycloak-instance.yml -n fapi 



sudo podman login -u $(oc whoami) -p $(oc whoami -t) image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com

sudo podman push localhost/kc-sig-fapi_resource_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_resource_server

sudo podman login -u $(oc whoami) -p $(oc whoami -t) image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com

sudo podman push localhost/kc-sig-fapi_client_jwks_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_client_jwks_server

sudo podman push localhost/kc-sig-fapi_api_gateway_nginx image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_api_gateway_nginx

sudo podman push localhost/kc-sig-fapi_server image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_server

sudo podman push localhost/kc-sig-fapi_keycloak image-registry-openshift-image-registry.apps.cluster-c767.dynamic.opentlc.com/openshift/kc-sig-fapi_keycloak
```