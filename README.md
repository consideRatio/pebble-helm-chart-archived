# Pebble Helm chart - Let's Encrypt for unreachable CI environment with Kubernetes!

[![TravisCI (.com) build status](https://img.shields.io/travis/com/jupyterhub/pebble-helm-chart/master?logo=travis)](https://travis-ci.com/jupyterhub/pebble-helm-chart)
[![Latest stable release of the Helm chart](https://img.shields.io/badge/dynamic/json.svg?label=version&url=https://jupyterhub.github.io/helm-chart/info.json&query=$.pebble.stable&colorB=orange&logo=helm)](https://jupyterhub.github.io/helm-chart/#pebble)
[![GitHub](https://img.shields.io/badge/issue_tracking-github-blue?logo=github)](https://github.com/jupyterhub/pebble-helm-chart/issues)
[![Discourse](https://img.shields.io/badge/help_forum-discourse-blue?logo=discourse)](https://discourse.jupyter.org/c/jupyterhub)
[![Gitter](https://img.shields.io/badge/social_chat-gitter-blue?logo=gitter)](https://gitter.im/jupyterhub/jupyterhub)


> __WARNING__: Pebble as an ACME server and this Helm chart is _only_ meant for testing purposes, it is _not secure_ and _not meant for production_.

[Pebble](https://github.com/letsencrypt/pebble) is an [ACME](https://letsencrypt.org/docs/glossary/#def-ACME) server like [Let's Encrypt](https://letsencrypt.org/). ACME servers can provide [TLS](https://letsencrypt.org/docs/glossary/#def-TLS) [certificates](https://letsencrypt.org/docs/glossary/#def-certificate) for HTTP over TLS ([HTTPS](https://en.wikipedia.org/wiki/HTTPS)) to [ACME clients](https://letsencrypt.org/docs/client-options/) that are able to prove control over a domain name through an [ACME challenge](https://letsencrypt.org/docs/challenge-types/).

This [Helm chart](https://helm.sh/docs/topics/charts/) makes it easy to install Pebble in a [Kubernetes cluster](https://kubernetes.io/) using [Helm](https://helm.sh/).

## Motivation

To test interactions against an ACME server like Let's Encrypt from an _unreachable_ CI environment like most ephemeral CI environments are, using [Let's Encrypts staging environment](https://letsencrypt.org/docs/staging-environment/) likely won't work, at least if you are using the HTTP-01 ACME challenge.

In the commonly used [HTTP-01 ACME challenge](https://letsencrypt.org/docs/challenge-types/#http-01-challenge), an ACME client proves its control of a domain's web server. During this challenge, the ACME server will lookup the domain name's IP and make a web request to it, and that's the problem! In an ephemeral CI environment, it is likely impossible to receive new incoming connections from Let's Encrypt's servers.

[![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIGF1dG9udW1iZXJcbiAgcGFydGljaXBhbnQgV2ViIFNlcnZlclxuICBwYXJ0aWNpcGFudCBBQ01FIENsaWVudFxuICBwYXJ0aWNpcGFudCBBQ01FIFNlcnZlclxuICBwYXJ0aWNpcGFudCBETlMgU2VydmVyXG4gIFxuICBhY3RpdmF0ZSBBQ01FIENsaWVudFxuXHRBQ01FIENsaWVudCAtPj4gQUNNRSBTZXJ2ZXI6IEknbSBkb2dzLmluZm8hXG4gIGRlYWN0aXZhdGUgQUNNRSBDbGllbnRcbiAgYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgQUNNRSBTZXJ2ZXIgLS0-PiBBQ01FIENsaWVudDogUHJvb3ZlIGl0ISBTYXkgbWVlZW9vdyFcbiAgZGVhY3RpdmF0ZSBBQ01FIFNlcnZlclxuICBhY3RpdmF0ZSBBQ01FIENsaWVudFxuICBBQ01FIENsaWVudCAtLT4-IFdlYiBTZXJ2ZXI6IC4uLiB3ZSBnb3QgdG8gbWVlZW9vdyAuLi5cbiAgZGVhY3RpdmF0ZSBBQ01FIENsaWVudFxuICBOb3RlIHJpZ2h0IG9mIEFDTUUgU2VydmVyOiBJbmRlcGVuZGVudCBsb29rdXAhXG5cdEFDTUUgU2VydmVyIC0-PiBETlMgU2VydmVyOiBkb2dzLmluZm8_XG4gIGRlYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgYWN0aXZhdGUgRE5TIFNlcnZlclxuICBETlMgU2VydmVyIC0-PiBBQ01FIFNlcnZlcjogMTMuMzcuMTMuMzchXG4gIGRlYWN0aXZhdGUgRE5TIFNlcnZlclxuICBhY3RpdmF0ZSBBQ01FIFNlcnZlclxuICBOb3RlIGxlZnQgb2YgQUNNRSBTZXJ2ZXI6IE5ldyBjb25uZWN0aW9uIVxuICBBQ01FIFNlcnZlciAtPj4gV2ViIFNlcnZlcjogSGkgZG9ncy5pbmZvIVxuICBkZWFjdGl2YXRlIEFDTUUgU2VydmVyXG4gIGFjdGl2YXRlIFdlYiBTZXJ2ZXJcbiAgV2ViIFNlcnZlciAtPj4gQUNNRSBTZXJ2ZXI6IG1lZWVvb3dcbiAgZGVhY3RpdmF0ZSBXZWIgU2VydmVyXG4gIGFjdGl2YXRlIEFDTUUgU2VydmVyXG4gIEFDTUUgU2VydmVyIC0-PiBBQ01FIENsaWVudDogVExTIENlcnRpZmljYXRlISBGZXRjaCFcbiAgZGVhY3RpdmF0ZSBBQ01FIFNlcnZlclxuIiwibWVybWFpZCI6eyJ0aGVtZSI6ImZvcmVzdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG4gIGF1dG9udW1iZXJcbiAgcGFydGljaXBhbnQgV2ViIFNlcnZlclxuICBwYXJ0aWNpcGFudCBBQ01FIENsaWVudFxuICBwYXJ0aWNpcGFudCBBQ01FIFNlcnZlclxuICBwYXJ0aWNpcGFudCBETlMgU2VydmVyXG4gIFxuICBhY3RpdmF0ZSBBQ01FIENsaWVudFxuXHRBQ01FIENsaWVudCAtPj4gQUNNRSBTZXJ2ZXI6IEknbSBkb2dzLmluZm8hXG4gIGRlYWN0aXZhdGUgQUNNRSBDbGllbnRcbiAgYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgQUNNRSBTZXJ2ZXIgLS0-PiBBQ01FIENsaWVudDogUHJvb3ZlIGl0ISBTYXkgbWVlZW9vdyFcbiAgZGVhY3RpdmF0ZSBBQ01FIFNlcnZlclxuICBhY3RpdmF0ZSBBQ01FIENsaWVudFxuICBBQ01FIENsaWVudCAtLT4-IFdlYiBTZXJ2ZXI6IC4uLiB3ZSBnb3QgdG8gbWVlZW9vdyAuLi5cbiAgZGVhY3RpdmF0ZSBBQ01FIENsaWVudFxuICBOb3RlIHJpZ2h0IG9mIEFDTUUgU2VydmVyOiBJbmRlcGVuZGVudCBsb29rdXAhXG5cdEFDTUUgU2VydmVyIC0-PiBETlMgU2VydmVyOiBkb2dzLmluZm8_XG4gIGRlYWN0aXZhdGUgQUNNRSBTZXJ2ZXJcbiAgYWN0aXZhdGUgRE5TIFNlcnZlclxuICBETlMgU2VydmVyIC0-PiBBQ01FIFNlcnZlcjogMTMuMzcuMTMuMzchXG4gIGRlYWN0aXZhdGUgRE5TIFNlcnZlclxuICBhY3RpdmF0ZSBBQ01FIFNlcnZlclxuICBOb3RlIGxlZnQgb2YgQUNNRSBTZXJ2ZXI6IE5ldyBjb25uZWN0aW9uIVxuICBBQ01FIFNlcnZlciAtPj4gV2ViIFNlcnZlcjogSGkgZG9ncy5pbmZvIVxuICBkZWFjdGl2YXRlIEFDTUUgU2VydmVyXG4gIGFjdGl2YXRlIFdlYiBTZXJ2ZXJcbiAgV2ViIFNlcnZlciAtPj4gQUNNRSBTZXJ2ZXI6IG1lZWVvb3dcbiAgZGVhY3RpdmF0ZSBXZWIgU2VydmVyXG4gIGFjdGl2YXRlIEFDTUUgU2VydmVyXG4gIEFDTUUgU2VydmVyIC0-PiBBQ01FIENsaWVudDogVExTIENlcnRpZmljYXRlISBGZXRjaCFcbiAgZGVhY3RpdmF0ZSBBQ01FIFNlcnZlclxuIiwibWVybWFpZCI6eyJ0aGVtZSI6ImZvcmVzdCJ9LCJ1cGRhdGVFZGl0b3IiOmZhbHNlfQ)



## Installation

### Standalone installation (recommended)
```shell
helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
helm install pebble jupyterhub/pebble
```

### Sub-chart installation
A packaged Helm chart contain the sub-charts Helm templates, and is therefore not recommended for Helm charts being packaged for distribution.

Installing Pebble as part of another chart should likely be made conditional using [_tags_ or _conditions_](https://helm.sh/docs/topics/charts/#tags-and-condition-fields-in-dependencies).

```yaml
# Chart.yaml - Helm 3 only, see note below for Helm 2 use.
apiVersion: v2
name: my-chart
# ...
dependencies:
  - name: pebble
    version: 0.1.0
    repository: https://jupyterhub.github.io/helm-chart/
    tags:
      - ci
```

> __NOTE:__ Helm 3 support `Chart.yaml` files with `apiVersion: v2`, and there you can specify chart dependencies directly. If you want to remain compatible with Helm 2 your `Chart.yaml` file has to have `apiVersion: v1` and the chart dependencies need to be specified in a separate `requirements.yaml` file.



## Helm chart configuration

### Configuring Helm charts
Helm charts render templates into Kubernetes yaml files using configurable values. A Helm chart comes with default values, and these can be overridden during chart installation and upgrades, for example with the `--values` flag to pass a [YAML](https://www.youtube.com/watch?v=cdLNKUoMc6c) file or with the `--set` flag.

To configure the Pebble Helm chart, create a `my-values.yaml` file to pass with `--values`. If you have installed it as a sub-chart, you should [nest the configuration](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/#overriding-values-from-a-parent-chart).

### Pebble configuration

#### Mischievous behavior?
Pebble is developed to test ACME clients and ensure they are robust, so it can intentionally act mischievous.

The default of this Helm chart seen below configures Pebble to ensure speedy certificate acquisition. Note that if you provide an array to `pebble.env`, it will override the default array of environment variables.

```yaml
pebble:
  env:
    # ref: https://github.com/letsencrypt/pebble#testing-at-full-speed
    - name: PEBBLE_VA_NOSLEEP
      value: "1"
```

See [Pebble's documentation](https://github.com/letsencrypt/pebble#testing-at-full-speed) for more info about its mischievous behavior.

#### Custom challenge ports
Pebble will connect with a domain's web-server during HTTP-01 (80) and TLS-ALPN-01 (443) challenges with specific ports, and you can configure those. This is useful if your web-server is behind a Kubernetes service exposing it on port 8080 for example.

```yaml
pebble:
  config:
    pebble:
      httpPort: 80 # this is the port where outgoing HTTP-01 challenges go
      tlsPort: 443 # this is the port where outgoing TLS-ALPN-01 challenges go
```

## ACME Client configuration

The [ACME client](https://letsencrypt.org/docs/client-options/) should be configured to work against the Pebble ACME server. The ACME client also needs to explicitly trust a root TLS certificates that has signed the leaf TLS certificate used by Pebble for the ACME communication which will be made over HTTPS.

### 1. URL to Pebble
The ACME client should communicate with something like `https://pebbles-service-name.pebbles-namespace/dir`. The namespace part can be omitted if Pebble is in the same namespace as the ACME client, and `pebbles-service-name` can be found with `kubectl get svc --all-namespaces | grep pebble`.

### 2. Trust Pebble's root TLS certificate
> __WARNING:__ All HTTPS communication should be treated as unsafe HTTP communication! This is only meant for testing!

The ACME client and anything communicating with Pebble's actual ACME Server or [management REST API](https://github.com/letsencrypt/pebble#management-interface) needs to trust [this root certificate](pebble/files/root-cert.pem). Its associated [_publicly exposed key_](pebble/files/root-key.pem) has signed the leaf certificate Pebble will use the HTTPS communication on port 443 (Pebble's ACME server) and 8080 (Pebble's REST API with `/roots/0` etc.).

[![](https://mermaid.ink/img/eyJjb2RlIjoiZ3JhcGggVERcbiAgc3ViZ3JhcGggUGViYmxlLUNoYWxsdGVzdHNydlxuICBkbnMoRE5TKVxuICBkbnMtbWdtdChETlMgTWFuYWdlbWVudClcbiAgZW5kXG5cbiAgc3ViZ3JhcGggUGViYmxlXG4gIGFjbWUoQUNNRSkgLi0-fHVkcCt0Y3A6Ly9wZWJibGU6ODA1M3wgZG5zXG4gIGFjbWUtbWdtdChBQ01FIE1hbmFtZ2VtZW50KVxuICBlbmRcbiAgXG4gIGNsaWVudCAtLT58aHR0cHM6Ly9wZWJibGU6ODQ0M3wgYWNtZVxuICBjbGllbnQoQUNNRSBjbGllbnQpXG4gIGRldihEZXZlbG9wZXIgLyBDSSlcbiAgZGV2IC0tPnxodHRwczovL3BlYmJsZTo4MDgwfCBhY21lLW1nbXRcbiAgZGV2IC4tPnxodHRwOi8vcGViYmxlOjgwODF8IGRucy1tZ210XG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZm9yZXN0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoiZ3JhcGggVERcbiAgc3ViZ3JhcGggUGViYmxlLUNoYWxsdGVzdHNydlxuICBkbnMoRE5TKVxuICBkbnMtbWdtdChETlMgTWFuYWdlbWVudClcbiAgZW5kXG5cbiAgc3ViZ3JhcGggUGViYmxlXG4gIGFjbWUoQUNNRSkgLi0-fHVkcCt0Y3A6Ly9wZWJibGU6ODA1M3wgZG5zXG4gIGFjbWUtbWdtdChBQ01FIE1hbmFtZ2VtZW50KVxuICBlbmRcbiAgXG4gIGNsaWVudCAtLT58aHR0cHM6Ly9wZWJibGU6ODQ0M3wgYWNtZVxuICBjbGllbnQoQUNNRSBjbGllbnQpXG4gIGRldihEZXZlbG9wZXIgLyBDSSlcbiAgZGV2IC0tPnxodHRwczovL3BlYmJsZTo4MDgwfCBhY21lLW1nbXRcbiAgZGV2IC4tPnxodHRwOi8vcGViYmxlOjgwODF8IGRucy1tZ210XG4iLCJtZXJtYWlkIjp7InRoZW1lIjoiZm9yZXN0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)

#### Avoid confusion: there are two root certificates
The other root certificate is what Pebble use to sign certificates for its ACME clients. Pebble recreate this root certificate on startup and expose it and its associated key through the management REST API on `https://pebble:8080/roots/0` without any authorization.

### Example ACME client configuration
The ACME client will need the root certificate to trust, and be configured to trust it.

#### Creating or re-using a ConfigMap
A Kubernetes ConfigMap can contain the root certificate to trust, and then be mounted as a file in the ACME client's pod's container.

If the Pebble Helm chart is installed in the ACME client's namespace, we can reuse a ConfigMap from it that contains the root certificate to trust. The ConfigMap's name can be found with `kubectl get cm --all-namespaces | grep pebble`.

Otherwise, you can create create a ConfigMap with the root certificate like this.

```yaml
cat <<EOF | kubectl apply --namespace <namespace-of-acme-client> -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: pebble
data:
  root-cert.pem: |
    -----BEGIN CERTIFICATE-----
    MIIDSzCCAjOgAwIBAgIIOvR7X+wFgKkwDQYJKoZIhvcNAQELBQAwIDEeMBwGA1UE
    AxMVbWluaWNhIHJvb3QgY2EgM2FmNDdiMCAXDTIwMDQyNjIzMDYxNloYDzIxMjAw
    NDI3MDAwNjE2WjAgMR4wHAYDVQQDExVtaW5pY2Egcm9vdCBjYSAzYWY0N2IwggEi
    MA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQCaoISLUOImo7vm7sGUpeycouDP
    TcJj6CxfCbvBsrlAg8ERGIph9H7TuDnTVk46pOaoxByGlwvvh4qR/Dled+G8NCt5
    s0r0yemY/fx1grm1TmcJRO+A1P5kx/M9hy+kVcyLRvPOnvo8Thj/4zvaJDh+pSjt
    5oAQvOHt9hYwGkkvSsZw12cTUuCsbypQ4lapDSeAjp3pNlqFcWmCvF9Ib3URDybN
    JWhY6yQQe54D2LxYqxCfYZjKhNbaxlNTlHu0Ujy75I/AdSjK6DljAZh0OimuQNEm
    FyXWvpnfyHbV5f0mMiXIOo2FY8izSD7cyFagmr0XvymCtxeDK1+MvT2pM+rXAgMB
    AAGjgYYwgYMwDgYDVR0PAQH/BAQDAgKEMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggr
    BgEFBQcDAjASBgNVHRMBAf8ECDAGAQH/AgEAMB0GA1UdDgQWBBR0MgecNe4RY575
    qAtIt6zAjbBqLTAfBgNVHSMEGDAWgBR0MgecNe4RY575qAtIt6zAjbBqLTANBgkq
    hkiG9w0BAQsFAAOCAQEAjtGjoXGRG7586vyT3XcJBa8y9MOsDhQGOec23h40NJCn
    SPF28bmTIaWhB+Hv8G+Mkyf9Ov3L5L/mH0VGvZUkMAnSdT4vaMYGrTvMtYGS/8ew
    lPnlSJ3oO9Kz9zfOneoPDD1OGkV0Oq3wLn9cq6jQgItEeACsXNtaogXJxYhvxiV1
    1k/gjXmG9pvFpb0A1bw6apxGftIViDKrPR2P/pG3QAuLKywQiNxZ5odf3kvKdZmJ
    hLbu119My9XiiWhNegufcRNRNEnKJ5AQsBEwLEnD4oeIZmFvYVKOPjfWRV5qczVi
    mUPjtQv88HhlgX/lBVWJ2VONlFWVoOreZz4GkAm5bA==
    -----END CERTIFICATE-----
EOF
```

#### Mounting from a ConfigMap
```yaml
# ... within a Pod specification
volumes:
  - name: pebble-root-cert
    configMap:
      name: pebble
      ## ... if the Pebble chart was installed as a sub-chart.
      #name: {{ .Release.Name }}-pebble
containers:
  - name: my-container-with-an-acme-client
    # ...
    volumeMounts:
      - name: pebble-root-cert
        subPath: root-cert.pem
        mountPath: /etc/pebble/root-cert.pem
```

#### Using the root certificate
Configuring the ACME client to trust a certain provided root certificate will depend on the ACME client. But as an example, a popular ACME client in Kubernetes contexts is [LEGO](https://github.com/go-acme/lego). LEGO can be configured to trust a root certificate and its signed leaf certificates if a file path is provided through the `LEGO_CA_CERTIFICATES` environment variable.

```yaml
# ... within a Pod specification template of a Helm chart
containers:
  - name: my-container-with-a-lego-acme-client
    # ...
    env:
      - name: LEGO_CA_CERTIFICATES
        value: /etc/pebble/root-cert.pem
```



## DNS configuration

Pebble as an ACME server needs to resolve domain names to where the ACME client can receive traffic. This can be done in various ways, and it is not apparent what makes the most sense for your setup. We recommend the basic or intermediary options below.

### Basic

If you don't need your ACME client to have a specific domain name (`mydomain.test`), you could test against the domain name of the ACME client's Kubernetes Service (`mysvc.mynamespace`). For example, if an ACME client is running in a pod targeted by the Kubernetes service called `client-svc` in the namespace `client-namespace`, then you could use `client-svc` and/or `client-svc.client-namespace` as domain names.

An upside of this approach is that any pod in Kubernetes will be able to find its to the actual web-server using the domain name, and not only those like Pebble using the configurable DNS server.

A downside of this approach is that requests must go to this specific domain name as it is the only reference to the Kubernetes service. This can be resolved

### Intermediary - Configuring the Kubernetes cluster's DNS server

You can configure the Kubernetes wide DNS server to point a set of domain names to this Kubernetes Service's domain name with CNAME records. While [kube-dns](https://github.com/kubernetes/dns) was common before, [CoreDNS](https://coredns.io/) is a common DNS server today.

CoreDNS is configured through a [Corefile](https://coredns.io/manual/toc/#configuration), which could very well be mounted through a configmap called `coredns` with a key called `Corefile` in the `kube-system` namespace. If that is the case, we can simply modify this ConfigMap and wait a while and then we will see the change.

Here is a Corefile part of the `corefile` configmap in `kube-system` that I got as part of starting a [k3s](https://k3s.io/) Kubernetes cluster.

```
.:53 {
    errors
    health
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
      pods insecure
      upstream
      fallthrough in-addr.arpa ip6.arpa
    }
    hosts /etc/coredns/NodeHosts {
      reload 1s
      fallthrough
    }
    prometheus :9153
    forward . /etc/resolv.conf
    cache 30
    loop
    reload
    loadbalance
}
```

By modifying this Corefile to have the following section below the line with "ready", we will make all lookups of `any.thing.test` resolve to the IP of `client-svc.client-namespace.svc.cluster.local`.

```
    template ANY ANY test {
      answer "{{ .Name }} 60 IN CNAME client-svc.client-namespace.svc.cluster.local"
    }
```

It is possible to do this modification with `kubectl edit configmap -n kube-system corefile` but also by a command which is suitable in a script for CI environments.

```
kubectl get configmap -n kube-system coredns -o jsonpath='{.data.Corefile}' \
| sed '/ready$/a \
    template ANY ANY test {\
      answer "{{ .Name }} 60 IN CNAME client-svc.client-namespace.svc.cluster.local"\
    }' \
> /tmp/Corefile

kubectl create configmap -n kube-system coredns --from-file Corefile=/tmp/Corefile -o yaml --dry-run=client \
| kubectl apply -f -
```



## Local development

### Prerequisites
- [docker](https://docs.docker.com/get-docker/)
- [k3d](https://github.com/rancher/k3d)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [helm](https://helm.sh/docs/intro/install/)

### Setup
```shell
# clone the git repo
git clone https://github.com/jupyterhub/pebble-helm-chart.git
cd pebble-helm-chart
```

```shell
# setup a local k8s cluster
k3d create --wait 60 --publish 8443:32443 --publish 8080:32080 --publish 8053:32053/udp --publish 8053:32053/tcp
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
```

```shell
# install pebble
helm upgrade --install pebble ./pebble --cleanup-on-fail
```

### Test
```shell
# run a basic health check
helm test pebble

kubectl logs pebble-test --all-containers
kubectl logs pebble-coredns-test --all-containers
```



## Release

No changelog or similar yet.

Making a release is as easy as pushing a tagged commit on the master branch.

```
git tag -a x.y.z -m x.y.z
git push --follow-tags
```
