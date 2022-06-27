[[Gitpod]]
```
#cat issuer.yml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: le-global-issuer
spec:
  acme:
    email: goran@crawlyfi.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-key
    solvers:
    - dns01:
        cloudflare:
          email: goran@crawlyfi.com
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

```
#cat secret.yml
---
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
  namespace: cert-manager
type: Opaque
stringData:
  api-token: mbsYjwURKCxXdw0qU5djJPDTieB23G3zig-lJ6hs
```