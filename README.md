```sh
# deploy argo in k8s
helm upgrade -i init . -n argocd --create-namespace

# สร้าง bcrypt hash ของรหัสผ่าน สำหรับ login admin (argocdServerAdminPassword)
htpasswd -nbBC 10 "" <password> | tr -d ':\n'

# deploy bootstrap in k8s
helm upgrade -i bootstrap ./02-bootstrap -n argocd --create-namespace
```
