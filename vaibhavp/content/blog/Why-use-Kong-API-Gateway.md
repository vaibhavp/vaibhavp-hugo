---
title: "Why Use Kong API Gateway"
date: 2023-03-28T10:11:26+05:30
draft: true
---

# Introduction

Kong API Gateway is a cloud-native API gateway that is based on the Nginx reverse proxy. It is a simple, fast, and lightweight solution that enables you to control, set up, and direct application-to-dedicated server request routing. The Kong API Gateway helps regulate who can access the services and data that are managed behind it. It ensures data security by allowing only authorized users and apps to access the data. The Kong API Gateway is highly performant and offers the following features:

- **Request/Response Transformation**: Kong can transform incoming and outgoing API requests and responses to conform to specific formats.
- **Authentication and Authorization**: Kong supports various authentication methods, including API key, OAuth 2.0, and JWT, and can enforce authorization policies for APIs.
- **Traffic Management**: Kong provides traffic management features, such as rate limiting, request throttling, and IP whitelisting, to maintain the reliability and stability of APIs.
- **Monitoring and Logging**: Kong offers detailed metrics and logs to help monitor API performance and identify issues.
- **Plugins**: Kong has a vast and continuously growing ecosystem of plugins that provide additional functionality, such as security, transformations, and integrations with other tools.
- **Microservice Architecture**: Kong is designed to work with microservice architecture, providing a central point of control for API traffic and security.
- **Scalability**: Kong is designed to scale horizontally, allowing it to handle large amounts of API traffic.

# Advantages of Using Kong as an API Gateway

Kong API Gateway is an efficient solution for managing APIs that offers advanced routing and management capabilities. With flexible request routing, automatic service discovery, advanced load balancing, comprehensive API management, and real-time analytics and monitoring, organizations can effectively route API traffic, discover and register APIs, distribute traffic across backend services, manage APIs throughout their lifecycle, and gain insight into API performance and usage. These capabilities make Kong a highly effective solution for managing APIs at scale and are essential for organizations looking to build and maintain a robust API infrastructure.

One of the key benefits of Kong is access to a wide range of plugins that can be easily added to the gateway, such as authentication, rate limiting, and transformations.

Another advantage of Kong is its flexible deployment options, which allow it to be deployed on-premises, in the cloud, or as a managed service, depending on the organization's needs. Additionally, Kong API Gateway provides improved security with features like authentication and authorization, encryption, and rate limiting that help protect sensitive data and prevent attacks on APIs.

# Use Cases for Kong

Kong API Gateway can be used for the following purposes:

- **API Gateway**: To manage and orchestrate microservices, providing a centralized management layer for APIs and ensuring that requests are routed to the appropriate services.
- **API security**: To provide robust security features, such as authentication, authorization, and encryption, which can be used to secure APIs and protect sensitive data.
- **Rate limiting and traffic control**: To control the rate at which API requests are processed, preventing APIs from becoming overwhelmed under heavy load.
- **Analytics and monitoring**: To provide real-time analytics and monitoring capabilities, including request and response logging, API usage tracking, and error reporting, which can be used to gain insight into API performance and usage.
- **Service discovery and load balancing**: To support advanced load balancing algorithms, such as round-robin, least connections, and IP hashing, enabling organizations to distribute API traffic across multiple backend services for improved reliability and performance.

# Getting started with Kong

To install the Kong API gateway and experience it, we will use our local Kubernetes cluster set up with [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/). Alternatively, you can use [minikube](https://minikube.sigs.k8s.io/docs/start/) to set up your local Kubernetes cluster.

## Installing Kong API Gateway On Kind Cluster

### Create Kind Cluster

- Create a kind cluster using the below configuration in your local system.

```bash
bash -c "cat <<EOF > /tmp/kind-config.yaml && kind create cluster --config /tmp/kind-config.yaml
apiVersion: kind.x-k8s.io/v1alpha4
kind: Cluster
name: kong-quick-start
networking:
  apiServerAddress: "0.0.0.0"
  apiServerPort: 16443
nodes:
  - role: control-plane
    extraPortMappings:
    - listenAddress: "0.0.0.0"
      protocol: TCP
      hostPort: 80
      containerPort: 80
    - listenAddress: "0.0.0.0"
      protocol: TCP
      hostPort: 443
      containerPort: 443
EOF"
```

- Set your kubeconfig context to use the kind-quick-start cluster

```bash
$ kubectl config use-context kind-kong-quick-start
$ kubectl cluster-info
```

- Create Kong Gateway secrets which are required for installation.

```bash
$ kubectl create namespace kong
$ kubectl create secret generic kong-config-secret -n kong \
    --from-literal=portal_session_conf='{"storage":"kong","secret":"super_secret_salt_string","cookie_name":"portal_session","cookie_same_site":"off","cookie_secure":false}' \
    --from-literal=admin_gui_session_conf='{"storage":"kong","secret":"super_secret_salt_string","cookie_name":"admin_session","cookie_same_site":"off","cookie_secure":false}' \
    --from-literal=pg_host="enterprise-postgresql.kong.svc.cluster.local" \
    --from-literal=kong_admin_password=kong \
    --from-literal=password=kong
$ kubectl create secret generic kong-enterprise-license --from-literal=license="'{}'" -n kong --dry-run=client -o yaml | kubectl apply -f -

```

- We will also install Cert Manager because kong uses Cert Manager to provide required certificates.

```bash
$ helm repo add jetstack https://charts.jetstack.io ; helm repo update
$ helm upgrade --install cert-manager jetstack/cert-manager --set installCRDs=true --namespace cert-manager --create-namespace
$ bash -c "cat <<EOF | kubectl apply -n kong -f -
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: quickstart-kong-selfsigned-issuer-root
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: quickstart-kong-selfsigned-issuer-ca
spec:
  commonName: quickstart-kong-selfsigned-issuer-ca
  duration: 2160h0m0s
  isCA: true
  issuerRef:
    group: cert-manager.io
    kind: Issuer
    name: quickstart-kong-selfsigned-issuer-root
  privateKey:
    algorithm: ECDSA
    size: 256
  renewBefore: 360h0m0s
  secretName: quickstart-kong-selfsigned-issuer-ca
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: quickstart-kong-selfsigned-issuer
spec:
  ca:
    secretName: quickstart-kong-selfsigned-issuer-ca
EOF"
```

- Now we will install Kong using it Helm chart.

```bash
$ helm repo add kong https://charts.konghq.com ; helm repo update
$ helm install quickstart kong/kong --namespace kong --values https://bit.ly/KongGatewayHelmValuesAIO
```

- Once all pods are running, try opening kong manager on your web browser using its domain. For example [https://kong.127-0-0-1.nip.io/](https://kong.127-0-0-1.nip.io/)

```bash
$ open "https://$(kubectl get ingress --namespace kong quickstart-kong-manager -o jsonpath='{.spec.tls[0].hosts[0]}')"
```

- You can also try to Curl Kong URL on your terminal.

```bash
$ curl --silent --insecure -X GET https://kong.127-0-0-1.nip.io/api -H 'kong-admin-token:kong'
```

# Conclusion

Kong API Gateway is a cloud-native API gateway that uses the Nginx reverse proxy to provide advanced routing and management capabilities. It helps organizations efficiently manage their APIs by offering flexible request routing, automatic service discovery, advanced load balancing, comprehensive API management, and real-time analytics and monitoring. Kong also provides improved security with features such as authentication and authorization, encryption, and rate limiting, as well as access to a large plugin ecosystem. Additionally, Kong can be deployed on-premises, in the cloud, or as a managed service, making it an effective solution for managing APIs at scale.