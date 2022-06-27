export INSTALL_K3S_EXEC="server --disable traefik --flannel-backend=none --node-label gitpod.io/workload_meta=true --node-label gitpod.io/workload_workspace_regular=true --node-label gitpod.io/workload_workspace_headless=true --node-label gitpod.io/workload_ide=true -node-label gitpod.io/workload_workspace_services=true"

curl -sfL https://get.k3s.io | sh -  

kubectl apply -f https://rancher.crawlyfi.com/v3/import/l2zskb5g6l86ldmkjlvng5l5gz78qj2gkgvx9gxp6dhnmxjmmqm7p6_c-m-9l2w8fnd.yaml

sudo cp /etc/rancher/k3s/k3s.yaml /home/gogsaan/.kube/config-files/

sudo cp kubernetes/k3s/calico-vsxlan.yaml /var/lib/rancher/k3s/server/manifests/