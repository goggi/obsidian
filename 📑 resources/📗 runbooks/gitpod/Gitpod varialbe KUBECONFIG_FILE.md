1. Get kubeconf file from rancher
2. Run in linux: ```cat kubeconfig | base64 -w 0 > kubeconfig_base64 ```
3. Run in linux: ``` nano kubeconfig_base64 ```
4. Copy file content to gitpod variable