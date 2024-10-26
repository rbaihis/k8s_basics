## list secrets && get info from secret 
- kubectl get secret  -A
- kubectl describe secret "secret_name or id" -n "name-space"
- data from a secret using "-o jsonpath='{ata.desiredField}'| base64 --decode"
  - Ex: (token) 
    - `kubectl describe secret "some-admin-token-secret" -n kube-system`   ==> Goal to know target aatibute  
    - `kubectl get secret "some-admin-token-secret" -n  -o jsonpath='{.data.token}' | base64 --decode`
## 
- 
