# ArgoCD Demo

คู่มือการเริ่มต้นใช้งาน ArgoCD สำหรับการจัดการ Kubernetes applications แบบ GitOps

## ขั้นตอนการติดตั้งและใช้งาน

### 1. ติดตั้ง ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. ตั้งค่า Ingress

ใช้ไฟล์ `01-init/argocd-ingress.yaml` เพื่อสร้าง Ingress:

```sh
kubectl apply -f 01-init/argocd-ingress.yaml
```

### 3. ปิด HTTPS ของ ArgoCD (แก้ปัญหา redirect loop)

เมื่อใช้ Ingress แล้ว ต้องปิด HTTPS ของ ArgoCD เพื่อหลีกเลี่ยง redirect loop:

```sh
kubectl -n argocd edit configmap argocd-cmd-params-cm
```

เพิ่มหรือแก้ไข:

```yaml
data:
  server.insecure: "true"
```

จากนั้น restart ArgoCD server:

```sh
kubectl -n argocd rollout restart deployment argocd-server
```

### 4. เข้าถึง ArgoCD Web UI

ดูรหัสผ่าน admin:

```sh
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 --decode
```

เข้าสู่ระบบด้วย:
- **Username:** `admin`
- **Password:** รหัสผ่านที่ได้จากคำสั่งด้านบน
- **URL:** เปิดผ่าน Ingress IP ที่ตั้งค่าไว้

### 5. สร้าง Bootstrap Application

Deploy bootstrap application เพื่อให้ ArgoCD จัดการ applications อื่นๆ ใน `03-apps` อัตโนมัติ:

```sh
helm upgrade -i bootstrap ./02-bootstrap -n argocd --create-namespace
```

หลังจากนี้ ArgoCD จะทำการ sync และจัดการ applications ใน `03-apps/applications/` ให้อัตโนมัติ

## โครงสร้างโปรเจค

- `01-init/` - ไฟล์สำหรับตั้งค่า ArgoCD (Ingress)
- `02-bootstrap/` - Bootstrap Helm chart สำหรับสร้าง Application ของ ArgoCD
- `03-apps/` - Applications และ Helm charts ที่จะถูกจัดการโดย ArgoCD