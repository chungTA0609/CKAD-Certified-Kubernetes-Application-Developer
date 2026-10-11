# KIẾN THỨC CKAD (Tổng hợp từ repository)

> Tài liệu tổng hợp tiếng Việt dùng như handbook ôn CKAD, biên soạn từ toàn bộ nội dung chính trong repo `chungTA0609/CKAD-Certified-Kubernetes-Application-Developer`.
>
> **Lưu ý nguồn:** Phần chiến lược/feedback exam bên dưới là tổng hợp từ tài liệu trong repo (bao gồm nguồn feedback NDA-compliant), **không phải tuyên bố quy định chính thức** từ CNCF/Linux Foundation.

## Mục lục

- [1) Cách dùng handbook này](#1-cách-dùng-handbook-này)
- [2) Bản đồ nội dung repo](#2-bản-đồ-nội-dung-repo)
- [3) Lộ trình học đề xuất](#3-lộ-trình-học-đề-xuất)
- [4) Quick reference kubectl](#4-quick-reference-kubectl)
- [5) Chủ đề CKAD cốt lõi (concept → mẫu → verify → gotchas → checklist)](#5-chủ-đề-ckad-cốt-lõi-concept-mẫu-verify-gotchas-checklist)
  - [5.1 Pod và resource requests/limits](#51-pod-và-resource-requestslimits)
  - [5.2 Multi-container và native sidecar (v1.35)](#52-multi-container-và-native-sidecar-v135)
  - [5.3 ConfigMap và Secret](#53-configmap-và-secret)
  - [5.4 RBAC](#54-rbac)
  - [5.5 NetworkPolicy (ingress/egress, AND/OR, DNS)](#55-networkpolicy-ingressegress-andor-dns)
  - [5.6 Deployment, rollout, rollback](#56-deployment-rollout-rollback)
  - [5.7 Service, selector, endpoints, port/targetPort](#57-service-selector-endpoints-porttargetport)
  - [5.8 Helm](#58-helm)
  - [5.9 Probes (liveness/readiness/startup)](#59-probes-livenessreadinessstartup)
  - [5.10 Ingress và Gateway API](#510-ingress-và-gateway-api)
  - [5.11 SecurityContext](#511-securitycontext)
  - [5.12 PV/PVC](#512-pvpvc)
  - [5.13 StatefulSet](#513-statefulset)
  - [5.14 DaemonSet](#514-daemonset)
  - [5.15 Init containers](#515-init-containers)
  - [5.16 In-place Pod Vertical Scaling](#516-in-place-pod-vertical-scaling)
  - [5.17 Jobs/CronJobs](#517-jobscronjobs)
- [6) Workflow troubleshooting tổng hợp](#6-workflow-troubleshooting-tổng-hợp)
- [7) Tổng hợp bộ luyện tập trong repo](#7-tổng-hợp-bộ-luyện-tập-trong-repo)
- [8) Scripts/Makefile: quy trình ôn tập thực chiến](#8-scriptsmakefile-quy-trình-ôn-tập-thực-chiến)
- [9) Điểm không nhất quán quan sát trong repo](#9-điểm-không-nhất-quán-quan-sát-trong-repo)
- [10) Chiến lược thi CKAD (theo tài liệu repo)](#10-chiến-lược-thi-ckad-theo-tài-liệu-repo)

## 1) Cách dùng handbook này

- Dùng theo flow: **nắm pattern → làm exercise → timed drills → mock exam → review weak areas**.
- Mỗi chủ đề bên dưới theo format cố định:
  1. Khái niệm cốt lõi
  2. YAML/kubectl tiêu biểu
  3. Verify
  4. Gotchas hay dính
  5. Checklist thực hành
- Khi ôn gấp: ưu tiên `patterns/`, `exercises/timed-exercises.md`, `mock-exams/`, `KUBECTL-CHEATSHEET.md`, `exam-strategy.md`.

## 2) Bản đồ nội dung repo

| Nhóm | Nội dung chính |
|---|---|
| Getting started | `README.md`, `START-HERE.md`, `INDEX.md`, `STUDY-PATHS.md` |
| 6 core patterns | `patterns/rbac-debug.md`, `deployment-fix.md`, `service-selector-mismatch.md`, `networkpolicy-allow-deny.md`, `cronjob-create.md`, `ingress-debug.md` |
| 14 bài hands-on | `exercises/01...14` |
| Scenario/tốc độ | `exercises/multi-step-scenarios.md`, `exercises/timed-exercises.md` |
| Mock exams + solutions | `mock-exams/MOCK-EXAM-01(.md/-SOLUTIONS)`, `MOCK-EXAM-02(.md/-SOLUTIONS)`, `FULL-MOCK-EXAM.md` |
| Troubleshooting | `troubleshooting/basics`, `networking`, `rbac`, `container`, `broken` |
| YAML templates | `skeletons/*.yaml` |
| Exam refs | `KUBECTL-CHEATSHEET.md`, `exam-strategy.md`, `EXAM-FEEDBACK-2026.md` |
| Quy trình tự luyện | `scripts/*.sh`, `Makefile` |

## 3) Lộ trình học đề xuất

### 1 tuần (crash course, 30–40h)
- Day 1–4: 6 patterns + timed tasks 2/5 phút.
- Day 5: multi-step scenarios + cheat sheet.
- Day 6: mock exam (2h strict timer).
- Day 7: review lỗi thường gặp + exam strategy.

### 2 tuần (50–60h)
- Tuần 1: pattern + exercises cơ bản + mock 01.
- Tuần 2: scenario nâng cao + troubleshooting + mock 02 + review.

### 3–4 tuần (80–100h)
- Tuần 1: pattern nền tảng.
- Tuần 2: toàn bộ exercises (01–14) + timed drills.
- Tuần 3: troubleshooting chuyên sâu + mock 01/02.
- Tuần 4 (tuỳ chọn): bù weak areas + polish speed/accuracy.

### Score target (theo STUDY-PATHS + mock docs)
- `<65%`: chưa nên thi.
- `65–75%`: có thể pass nhưng còn rủi ro.
- `75–85%`: vùng an toàn.
- `85%+`: rất tốt.

## 4) Quick reference kubectl

### Alias/biến môi trường
```bash
alias k=kubectl
export do='--dry-run=client -o yaml'
export now='--force --grace-period=0'
```

### Tạo resource nhanh (imperative)
```bash
k create deployment app --image=nginx
k expose deployment app --port=80 --target-port=8080 --name=app-svc
k create configmap app-config --from-literal=MODE=prod
k create secret generic db-creds --from-literal=username=admin --from-literal=******
k create serviceaccount app-sa -n prod
k create role pod-reader --verb=get,list --resource=pods -n prod
k create rolebinding read-bind --role=pod-reader --serviceaccount=prod:app-sa -n prod
k create cronjob backup --image=busybox --schedule='0 2 * * *' -- echo ok
```

### Inspect / debug
```bash
k get all -n <ns>
k get pod --show-labels -n <ns>
k describe pod <pod> -n <ns>
k logs <pod> -n <ns>
k logs <pod> -c <container> -n <ns>
k logs <pod> --previous -n <ns>
k exec -it <pod> -n <ns> -- sh
k get events -n <ns> --sort-by=.lastTimestamp
k get endpoints <svc> -n <ns>
k get <resource> -o jsonpath='{...}'
```

### Apply / patch / auth / context / explain
```bash
k apply -f file.yaml
k apply -f dir/
k apply --dry-run=client -o yaml -f file.yaml

k patch deployment app -n prod -p '{"spec":{"replicas":5}}'

k auth can-i list pods --as=system:serviceaccount:prod:app-sa -n prod

k config current-context
k config get-contexts
k config use-context <ctx>
k config set-context --current --namespace=<ns>

k explain deployment.spec.template.spec.containers
k explain networkpolicy --recursive
```

## 5) Chủ đề CKAD cốt lõi (concept → mẫu → verify → gotchas → checklist)

### 5.1 Pod và resource requests/limits

**Concept cốt lõi**
- `requests`: mức tài nguyên scheduler dùng để đặt Pod.
- `limits`: trần tài nguyên runtime.
- Lỗi rất thường gặp trong repo: nhầm requests/limits hoặc sai đơn vị CPU (`100m` vs `100`).

**Mẫu YAML**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
  - name: web
    image: nginx:1.25
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

**Verify**
```bash
k get pod web -n exercise-01 --show-labels
k describe pod web -n exercise-01 | grep -A 4 'Limits\|Requests'
```

**Gotchas**
- `kubectl run` không set đầy đủ resources theo đề.
- Quên namespace khiến verify fail.

**Checklist**
- [ ] Đúng namespace
- [ ] Đúng label
- [ ] Đủ request + limit, đúng đơn vị

### 5.2 Multi-container và native sidecar (v1.35)

**Concept cốt lõi**
- Sidecar pattern: container chính + container phụ dùng volume chung (`emptyDir`).
- Native sidecar (v1.35): khai báo trong `initContainers` với `restartPolicy: Always`.

**Mẫu YAML (native sidecar)**
```yaml
spec:
  initContainers:
  - name: log-agent
    image: busybox
    restartPolicy: Always
    volumeMounts:
    - name: logs
      mountPath: /var/log
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: logs
    emptyDir: {}
```

**Verify**
```bash
k get pod app-with-sidecar -n exercise-02
k logs app-with-sidecar -n exercise-02 -c log-agent
```

**Gotchas**
- Cả 2 container phải mount cùng volume.
- `emptyDir` mất dữ liệu khi Pod bị xoá.

**Checklist**
- [ ] Shared volume map đúng cả 2 phía
- [ ] Command sidecar chờ file/log hợp lý
- [ ] Biết phân biệt classic sidecar vs native sidecar

### 5.3 ConfigMap và Secret

**Concept cốt lõi**
- ConfigMap: config không nhạy cảm.
- Secret: dữ liệu nhạy cảm (repo dùng dạng demo).
- Có 2 cách inject phổ biến: `envFrom` và volume mount.

**Mẫu YAML (từ skeleton + exercise)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-app
spec:
  containers:
  - name: app
    image: nginx
    envFrom:
    - configMapRef:
        name: app-config
    env:
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: password
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/db-creds
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-creds
```

**Verify**
```bash
k exec config-app -n exercise-03 -- env | grep -E 'DB_HOST|LOG_LEVEL'
k exec config-app -n exercise-03 -- cat /etc/db-creds/username
```

**Gotchas**
- Secret trong YAML cần base64 (`data`) hoặc dùng `stringData`.
- `envFrom` load toàn bộ key, còn `valueFrom` load từng key.

**Checklist**
- [ ] Tách config và secret đúng mục đích
- [ ] Dùng readOnly cho secret volume
- [ ] Verify env + file mount

### 5.4 RBAC

**Concept cốt lõi**
- Bộ 3 hay gặp: `ServiceAccount` + `Role/ClusterRole` + `RoleBinding/ClusterRoleBinding`.
- Quyền phải verify bằng `kubectl auth can-i`.

**Lệnh mẫu nhanh (pattern rbac-debug)**
```bash
k create sa app-sa -n prod
k create role pod-reader --verb=get,list --resource=pods -n prod
k create rolebinding app-sa-binding --role=pod-reader --serviceaccount=prod:app-sa -n prod
k auth can-i list pods --as=system:serviceaccount:prod:app-sa -n prod
```

**Verify**
- Test cả trường hợp được phép và không được phép (`yes`/`no`).

**Gotchas**
- Sai namespace trong RoleBinding subject.
- Dùng sai format `--as=system:serviceaccount:<ns>:<sa>`.

**Checklist**
- [ ] SA có tồn tại
- [ ] Verb/resource đúng đề
- [ ] Binding đúng namespace
- [ ] Có verify yes/no

### 5.5 NetworkPolicy (ingress/egress, AND/OR, DNS)

**Concept cốt lõi**
- NetworkPolicy là **explicit allow**; phần không allow thì bị deny.
- Ingress rule gắn theo **pod bị bảo vệ** (`podSelector`).
- Repo nhấn mạnh rất nhiều: egress DNS (UDP 53) dễ bị quên.

**Mẫu YAML (allow ingress + DNS egress)**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-policy
spec:
  podSelector:
    matchLabels:
      role: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: db
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
```

**AND/OR cần nhớ**
- Nhiều item trong cùng danh sách `from` thường là logic OR theo peer rule.
- Nếu ghép selector sai cấu trúc, có thể tạo hiệu ứng không như mong muốn (feedback repo cảnh báo nhầm AND/OR rất phổ biến).
- Cách an toàn khi làm bài: tách rõ từng khối rule cho từng nguồn traffic, rồi verify bằng test pod.

**Verify**
```bash
k get netpol -n <ns>
k describe netpol <name> -n <ns>
k exec <source-pod> -n <ns> -- wget -O- http://<dest>:<port>
```

**Gotchas**
- Quên `policyTypes: Egress` nhưng lại khai báo `egress`.
- `podSelector: {}` chọn toàn bộ Pod trong namespace.
- CNI không hỗ trợ NetworkPolicy thì test sai kỳ vọng.

**Checklist**
- [ ] Xác định đúng pod đích được bảo vệ
- [ ] Đủ ingress/egress theo đề
- [ ] Có DNS egress nếu cần resolve tên
- [ ] Test được cả path allowed và denied

### 5.6 Deployment, rollout, rollback

**Concept cốt lõi**
- Chiến lược rolling update thường yêu cầu `maxSurge`/`maxUnavailable`.
- Rollback bằng `kubectl rollout undo`.
- Debug deployment: `describe` → `logs` → kiểm tra image/probe/config/resources.

**Lệnh mẫu**
```bash
k create deployment webapp --image=nginx:1.24 -n exercise-06
k scale deployment webapp --replicas=4 -n exercise-06
k set image deployment/webapp nginx=nginx:1.25 -n exercise-06
k rollout status deployment/webapp -n exercise-06
k rollout history deployment/webapp -n exercise-06
k rollout undo deployment/webapp -n exercise-06
```

**Verify**
```bash
k get deploy webapp -n exercise-06
k get deploy webapp -n exercise-06 -o jsonpath='{.spec.template.spec.containers[0].image}'
```

**Gotchas**
- Set lại cùng image sẽ không tạo revision mới.
- `maxSurge=0` và `maxUnavailable=0` gây deadlock rollout.

**Checklist**
- [ ] Replica + strategy đúng đề
- [ ] Rollout thành công
- [ ] Rollback quay về image mong muốn

### 5.7 Service, selector, endpoints, port/targetPort

**Concept cốt lõi**
- Lỗi số 1 trong repo: selector mismatch ⇒ endpoints rỗng.
- Ingress backend phải trỏ vào **service port**, không phải `targetPort`.

**Mẫu YAML**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

**Verify**
```bash
k get svc myapp-svc -o jsonpath='{.spec.selector}'
k get pod --show-labels
k get endpoints myapp-svc
```

**Gotchas**
- Sai 1 ký tự/case ở label là rớt endpoint.
- `Service` và Pod khác namespace sẽ không thấy nhau.

**Checklist**
- [ ] Selector match đúng label Pod
- [ ] Endpoint không rỗng
- [ ] Phân biệt rõ containerPort / service port / targetPort

### 5.8 Helm

**Concept cốt lõi**
- Repo luyện thao tác thực dụng: add repo, install, list, upgrade/rollback.
- Exam feedback trong repo nhấn mạnh Helm thực tế khó hơn mock cơ bản.

**Lệnh mẫu (exercise 07)**
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
k create ns exercise-07
helm install myredis bitnami/redis -n exercise-07
helm list -n exercise-07
```

**Verify**
```bash
helm status myredis -n exercise-07
k get pods -n exercise-07
```

**Gotchas**
- Release đã tồn tại: cân nhắc `helm upgrade --install`.
- Namespace có thể cần `--create-namespace`.

**Checklist**
- [ ] Repo/chart đúng
- [ ] Release chạy ổn
- [ ] Biết xem values/history/rollback

### 5.9 Probes (liveness/readiness/startup)

**Concept cốt lõi**
- Liveness fail: restart container.
- Readiness fail: không nhận traffic (không restart).
- Startup probe bảo vệ app startup chậm.

**Mẫu YAML**
```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  periodSeconds: 10
readinessProbe:
  tcpSocket:
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
startupProbe:
  httpGet:
    path: /
    port: 80
  failureThreshold: 30
  periodSeconds: 10
```

**Verify**
```bash
k describe pod probe-test -n exercise-08 | grep -A 3 'Liveness\|Readiness\|Startup'
```

**Gotchas**
- Probe quá gắt làm CrashLoopBackOff.
- Sai path/port probe khiến Pod never ready.

**Checklist**
- [ ] Chọn đúng loại probe theo yêu cầu
- [ ] Delay/threshold hợp lý với startup app
- [ ] Verify trạng thái READY và events

### 5.10 Ingress và Gateway API

**Concept cốt lõi**
- Ingress: route HTTP theo host/path.
- Gateway API: dùng `GatewayClass`, `Gateway`, `HTTPRoute`.
- Bài tập repo yêu cầu biết cả hai (exercise 09, skeleton gateway-api.yaml).

**Mẫu Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store-ingress
spec:
  rules:
  - host: store.example.com
    http:
      paths:
      - path: /shop
        pathType: Prefix
        backend:
          service:
            name: shop-svc
            port:
              number: 80
```

**Mẫu HTTPRoute (rút gọn)**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: app-route
spec:
  parentRefs:
  - name: main-gateway
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /api
    backendRefs:
    - name: api-backend
      port: 8080
```

**Verify**
```bash
k get ingress -n <ns>
k describe ingress <name> -n <ns>
k get httproute -n <ns>
k get svc,ep -n <ns>
```

**Gotchas**
- Ingress không có controller thì resource tồn tại nhưng không route được.
- Gateway API cần CRDs.
- Port ở ingress/httpRoute phải khớp service port.

**Checklist**
- [ ] Host/path đúng đề
- [ ] Backend service tồn tại, endpoint có IP
- [ ] Class/parentRefs đúng môi trường

### 5.11 SecurityContext

**Concept cốt lõi**
- Hardening thường gặp: `runAsNonRoot`, `runAsUser`, `fsGroup`, drop capabilities, disable privilege escalation, `readOnlyRootFilesystem`.
- Repo nhấn mạnh phải thêm writable volume (`emptyDir`) cho app cần ghi file tạm.

**Mẫu YAML (rút gọn)**
```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
containers:
- name: app
  image: nginx
  securityContext:
    allowPrivilegeEscalation: false
    capabilities:
      drop: ["ALL"]
    readOnlyRootFilesystem: true
```

**Verify**
```bash
k describe pod locked-down -n exercise-10
k exec locked-down -n exercise-10 -- id
```

**Gotchas**
- `readOnlyRootFilesystem` dễ làm app fail nếu thiếu mount writable.
- Nhầm vai trò giữa `runAsUser`, `runAsGroup`, `fsGroup`.

**Checklist**
- [ ] Cấu hình security phù hợp đề
- [ ] App vẫn chạy được với volume hỗ trợ ghi
- [ ] Không cấp capability dư thừa

### 5.12 PV/PVC

**Concept cốt lõi**
- PVC yêu cầu dung lượng/accessMode; trạng thái `Pending` nếu chưa bind.
- Repo dùng nhiều tình huống debug storageClass mismatch (đặc biệt bonus mock).

**Mẫu PVC**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 256Mi
```

**Verify**
```bash
k get pvc -n <ns>
k describe pvc data-pvc -n <ns>
```

**Gotchas**
- `ReadWriteOnce` là 1 node, không phải 1 pod.
- storageClassName không tồn tại ⇒ Pod/PVC pending.

**Checklist**
- [ ] accessModes + size đúng yêu cầu
- [ ] PVC đã Bound
- [ ] Pod mount đúng claimName

### 5.13 StatefulSet

**Concept cốt lõi**
- Cần headless service (`clusterIP: None`) và `serviceName` khớp.
- Pod identity ổn định (`db-0`, `db-1`, ...), có PVC riêng từng replica.

**Mẫu YAML (ý chính)**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: db
spec:
  serviceName: db-headless
  replicas: 3
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

**Verify**
```bash
k get sts db -n exercise-11
k get pods -n exercise-11 -l app=db
k get pvc -n exercise-11
k run dns-test --image=busybox --rm -it -n exercise-11 -- nslookup db-0.db-headless.exercise-11.svc.cluster.local
```

**Gotchas**
- Thiếu headless service hoặc `serviceName` sai sẽ hỏng DNS.
- Xoá StatefulSet không tự xoá PVC.

**Checklist**
- [ ] Headless service tạo trước/đúng tên
- [ ] Pod tạo theo thứ tự
- [ ] PVC per-pod đầy đủ

### 5.14 DaemonSet

**Concept cốt lõi**
- 1 Pod mỗi node phù hợp (log collector/agent).
- Control-plane node thường có taint, cần toleration nếu đề yêu cầu chạy trên đó.

**Mẫu YAML**
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-collector
spec:
  selector:
    matchLabels:
      app: log-collector
  template:
    metadata:
      labels:
        app: log-collector
    spec:
      tolerations:
      - key: node-role.kubernetes.io/control-plane
        effect: NoSchedule
      containers:
      - name: agent
        image: fluentd:v1.16-1
```

**Verify**
```bash
k get ds log-collector -n exercise-12
k get pods -n exercise-12 -o wide
k get nodes --no-headers | wc -l
```

**Gotchas**
- Không có trường `replicas` trong DaemonSet.
- Thiếu toleration ⇒ thiếu Pod ở control-plane.

**Checklist**
- [ ] Pod count ~ node count mục tiêu
- [ ] hostPath/volume mount đúng đường dẫn
- [ ] Resource request/limit đúng đề

### 5.15 Init containers

**Concept cốt lõi**
- Chạy tuần tự trước app container; phải exit code 0 mới qua bước sau.
- Hợp với use case chờ dependency/DNS, chuẩn bị config/schema.

**Mẫu YAML ý chính**
```yaml
initContainers:
- name: wait-for-db
  image: busybox
  command: ["sh", "-c", "until nslookup db-svc.exercise-13.svc.cluster.local; do echo waiting; sleep 2; done"]
containers:
- name: app
  image: nginx
```

**Verify**
```bash
k get pod web-app -n exercise-13
k logs web-app -n exercise-13 -c wait-for-db
```

**Gotchas**
- Init fail sẽ retry và Pod kẹt ở `Init:x/y`.
- Xem log phải chỉ định `-c <init-container>`.

**Checklist**
- [ ] Init order đúng mục đích
- [ ] Dependency (service/DNS/file) có tồn tại
- [ ] Main app chạy sau init hoàn tất

### 5.16 In-place Pod Vertical Scaling

**Concept cốt lõi**
- Từ v1.35: patch resources trên Pod đang chạy.
- `resizePolicy` quyết định cần restart hay không (CPU thường `NotRequired`, memory hay `RestartContainer`).

**Lệnh mẫu**
```bash
k patch pod resize-demo -n exercise-14 --patch '{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"cpu":"200m"},"limits":{"cpu":"400m"}}}]}}'
k patch pod resize-demo -n exercise-14 --patch '{"spec":{"containers":[{"name":"nginx","resources":{"requests":{"memory":"128Mi"},"limits":{"memory":"256Mi"}}}]}}'
```

**Verify**
```bash
k get pod resize-demo -n exercise-14 -o jsonpath='{.spec.containers[0].resources}'
k get pod resize-demo -n exercise-14 -o jsonpath='{.status.resize}'
k get pod resize-demo -n exercise-14 -o jsonpath='{.status.containerStatuses[0].restartCount}'
```

**Gotchas**
- Phải khai báo `resizePolicy` ngay từ lúc tạo Pod.
- Runtime/feature gate môi trường phải hỗ trợ.

**Checklist**
- [ ] CPU resize không restart khi policy cho phép
- [ ] Memory resize có thể restart đúng kỳ vọng
- [ ] Theo dõi `status.resize` và restart count

### 5.17 Jobs/CronJobs

**Concept cốt lõi**
- Job: chạy 1 lần; CronJob: chạy theo lịch.
- Cần chú ý cron format, history limits, concurrency policy.

**Mẫu YAML CronJob**
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 2 * * *"
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: job
            image: busybox
            command: ["sh", "-c", "echo backup"]
          restartPolicy: OnFailure
```

**Verify**
```bash
k get cronjob backup-job -n <ns>
k describe cronjob backup-job -n <ns>
k get jobs -n <ns>
```

**Gotchas**
- Sai cron expression hoặc quên UTC assumption.
- `restartPolicy` cho job template thường dùng `OnFailure`/`Never`, không phải `Always`.

**Checklist**
- [ ] Schedule hợp lệ
- [ ] Job template chạy được
- [ ] history/concurrency đúng đề

## 6) Workflow troubleshooting tổng hợp

### A. Container/Pod crash workflow
1. `k get pod -n <ns>` → xem STATUS/RESTARTS
2. `k describe pod <pod> -n <ns>` → events/probe/image error
3. `k logs <pod> -n <ns>` (+ `--previous` nếu restart)
4. Nếu cần: `k exec -it <pod> -- sh`
5. Sửa nhanh bằng `k set image`, `k edit`, hoặc apply YAML fix

### B. Networking workflow (service/ingress/netpol)
1. `k get svc,ep,ing -n <ns>`
2. Kiểm tra `selector` vs `pod labels`
3. Kiểm tra chain port: containerPort ↔ targetPort ↔ service port ↔ ingress backend port
4. Với NetworkPolicy: kiểm tra podSelector/policyTypes/ingress/egress + DNS UDP 53
5. Test từ source pod: `wget/curl/nc`

### C. RBAC workflow
1. Xác nhận SA có tồn tại
2. Liệt kê binding: RoleBinding/ClusterRoleBinding
3. Xem role verbs/resources
4. Verify bằng `k auth can-i ... --as=system:serviceaccount:<ns>:<sa>`
5. Nếu fail: sửa role hoặc binding namespace

### D. Checklist chống lỗi ngớ ngẩn (từ troubleshooting/basics)
- [ ] Đúng namespace/context trước mỗi thao tác
- [ ] Label/selector khớp exact (case-sensitive)
- [ ] Kiểm tra cả status + events + logs
- [ ] Không bỏ qua bước verify sau khi tạo/sửa resource

## 7) Tổng hợp bộ luyện tập trong repo

### 6 core patterns (xuất hiện lặp lại)
1. RBAC debug
2. Deployment fix
3. Service selector mismatch
4. NetworkPolicy allow/deny
5. CronJob create
6. Ingress debug

### 14 exercises (01–14)
- Bao phủ từ Pod cơ bản đến in-place scaling v1.35.
- Mỗi bài đều có: task, hints, gotchas, verify, cleanup, solution.

### Timed exercises
- Tier 2 phút / 5 phút / 10 phút.
- Có rubric điểm (đúng thời gian, quá giờ, lỗi logic).

### Multi-step scenarios
- 4 kịch bản 10–15 phút dạng “nhiều lớp lỗi/nhiều thành phần”.
- Tập kỹ năng phối hợp Deployment + Service + RBAC + NetworkPolicy + CronJob.

### Mock exams + solutions
- `MOCK-EXAM-01.md`: 15 câu + bonus debug.
- `MOCK-EXAM-02.md`: 16 câu + bonus debug.
- `FULL-MOCK-EXAM.md`: đề full có solution dạng details.
- 2 file `*-SOLUTIONS.md`: lời giải chi tiết + verify + pitfalls.

## 8) Scripts/Makefile: quy trình ôn tập thực chiến

### Scripts hữu ích
- `scripts/create-cluster.sh`: tạo cluster practice (`kind|minikube|k3d`, mặc định Kubernetes v1.35.0).
- `scripts/exam-setup.sh`: setup alias `k`, `do`, completion, vim YAML tiện cho thi.
- `scripts/quiz.sh`: quiz random theo domain/speed mode, có verify và score history.
- `scripts/mock-exam.sh`: mô phỏng exam 2h, chấm điểm theo domain, có skip/flag.

### Makefile workflow
```bash
make setup          # tạo cluster + exam setup
make quiz           # luyện nhanh theo câu hỏi terminal
make mock-exam      # thi thử timed
make verify EX=01   # verify một exercise
make verify-all     # verify toàn bộ exercise scripts
make docs           # chạy mkdocs local
make lint           # validate skeleton yaml (kubeconform nếu có)
```

## 9) Điểm không nhất quán quan sát trong repo

> Mục này chỉ ghi nhận để người học tự đối chiếu, **không sửa nội dung nguồn**.

1. **Metadata mkdocs khác fork hiện tại**: `mkdocs.yml` vẫn để `techwithmohamed/...` cho `site_url`, `repo_url`, `repo_name`.
2. **Số lượng mock exam không thống nhất**:
   - README ghi “1 full mock exam (15 questions)”.
   - Repo hiện có `MOCK-EXAM-01`, `MOCK-EXAM-02`, `FULL-MOCK-EXAM` và solutions.
3. **Số lượng câu mock không đồng nhất giữa tài liệu**:
   - `MOCK-EXAM-01.md`: 15 câu chính + bonus.
   - `MOCK-EXAM-02.md`: 16 câu chính + bonus.
   - `scripts/mock-exam.sh`: mô tả 17 questions.
4. **Một số liên kết có dấu hiệu lệch path/anchor**:
   - `STUDY-PATHS.md` có link `basics/COMMON-MISTAKES.md` (thiếu prefix `troubleshooting/`).
   - `START-HERE.md` phần 3-4 tuần trỏ anchor khác heading thực tế.
   - `exam-strategy.md` nhắc `/mock-exam/` (trong repo là `mock-exams/`).

## 10) Chiến lược thi CKAD (theo tài liệu repo)

### Format và áp lực thời gian
- Thời lượng: 2 giờ.
- Câu hỏi thực hành terminal-based, có trọng số.
- Mục tiêu pass: 66% (theo tài liệu repo).

### Two-pass strategy (khuyến nghị mạnh)
- **Pass 1:** quét câu dễ/trung bình, lấy điểm chắc.
- **Pass 2:** quay lại câu khó/đã flag.
- Rule thường dùng trong repo: kẹt >3 phút không tiến triển thì skip tạm.

### Khi nào nên skip
- Bị mắc syntax/debug loop kéo dài.
- Nghi ngờ đọc sai đề.
- Không có progress rõ sau vài phút.

### Checklist theo loại câu
- **RBAC:** SA → Role/ClusterRole → Binding → `auth can-i`.
- **Deployment:** image/probe/resources/rollout.
- **NetworkPolicy:** podSelector đích, ingress/egress explicit allow, DNS.
- **Service/Ingress:** endpoint non-empty, port chain đúng.
- **CronJob:** schedule + history + restartPolicy.

### Score targets thực dụng
- Mock `<65%`: chưa ổn.
- `65–75%`: có thể thi nhưng nên củng cố thêm.
- `75%+`: vùng sẵn sàng tốt.
- `80%+`: mức tự tin cao.

### Ghi chú nguồn feedback
- `EXAM-FEEDBACK-2026.md` nêu rõ là tổng hợp NDA-compliant từ cộng đồng, mang tính quan sát (sample-based), không phải đề thi chính thức.
- Điểm nhấn lặp lại trong feedback repo: NetworkPolicy (đặc biệt egress + DNS + logic rule), troubleshooting nhiều bước, Helm tăng độ hiện diện, Gateway API có xuất hiện.

---

Nếu bạn học theo repo này, khuyến nghị cuối cùng:
1. Thuộc 6 patterns ở mức có thể gõ lại từ trí nhớ.
2. Làm hết 14 exercises (ít nhất 1 lượt có verify).
3. Chạy timed drills + 2 mock exam trong điều kiện giới hạn thời gian.
4. Sau mỗi buổi, log lại lỗi lặp (namespace/context/selector/port/RBAC syntax) và sửa theo checklist ở trên.
