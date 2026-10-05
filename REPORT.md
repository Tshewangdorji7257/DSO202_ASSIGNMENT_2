# DSO202 — Assignment 2
## Kubernetes Application Deployment with Ingress

**Student:** Tshewang Dorji  
**Application:** Field Log — Task Tracker  
**Platform:** Kubernetes on kind  
**Namespace:** dso202-assignment-01  
**Ingress:** NGINX Ingress  
**Database:** PostgreSQL StatefulSet + PVC

---

## 1. Introduction

This assignment demonstrates the deployment of a multi-tier task-tracking
application on a local Kubernetes cluster created using kind.

The application consists of:
- Frontend
- Backend API
- PostgreSQL database

Kubernetes Services provide communication between the application
components, while NGINX Ingress provides HTTP routing through a single
entry point.

---

## 2. Objectives

The main objectives of this assignment were:

- Deploy the application components in Kubernetes.
- Configure Kubernetes Services.
- Deploy PostgreSQL using a StatefulSet.
- Configure persistent storage using a PVC.
- Install and configure NGINX Ingress.
- Route frontend and backend traffic through Ingress.
- Verify frontend, backend and database connectivity.
- Test the final application.

---

## 3. Final Kubernetes Environment

The final namespace contains:

- Frontend Pod
- Backend Pod
- PostgreSQL Pod
- Frontend NodePort Service
- Backend ClusterIP Service
- PostgreSQL Headless Service
- PostgreSQL PersistentVolumeClaim
- NGINX Ingress

### Evidence 1 — Final Kubernetes Resources

**Insert Screenshot Here**

```powershell
kubectl get pods,svc,pvc,ingress -n dso202-assignment-01
````

**Figure 1: Final Pods, Services, PVC and Ingress**

The screenshot should show all application components running and the
database PVC in the Bound state.

---

## 4. PostgreSQL StatefulSet and Persistent Storage

PostgreSQL was deployed using a StatefulSet named `db`.

The StatefulSet provides stable identity for the PostgreSQL Pod, while the
PersistentVolumeClaim provides persistent storage.

The final database configuration showed:

* StatefulSet: `db`
* Ready: `1/1`
* PVC: `data-db-0`
* Capacity: `1Gi`
* Access mode: `RWO`
* Status: `Bound`

### Evidence 2 — StatefulSet

**Insert Screenshot Here**

```powershell
kubectl get statefulset -n dso202-assignment-01
```

**Figure 2: PostgreSQL StatefulSet**

### Evidence 3 — PVC

**Insert Screenshot Here**

```powershell
kubectl get pvc -n dso202-assignment-01
```

**Figure 3: PostgreSQL PersistentVolumeClaim**

---

## 5. Backend Service

The backend API is exposed internally using the `backend-svc`
ClusterIP Service.

The Service listens on port `8080` and forwards traffic to the backend Pod.

### Evidence 4 — Backend Service and Endpoint

**Insert Screenshot Here**

```powershell
kubectl get svc backend-svc -n dso202-assignment-01

kubectl get endpoints backend-svc -n dso202-assignment-01
```

**Figure 4: Backend Service and Endpoint**

The endpoint confirmed that the backend Service was connected to the
running backend Pod.

---

## 6. NGINX Ingress Controller

NGINX Ingress was installed to provide external HTTP routing.

During installation, the controller initially experienced scheduling and
image-pull problems.

The control-plane node was labelled:

```powershell
kubectl label node control-plane ingress-ready=true
```

The controller image initially experienced a TLS handshake timeout when
pulling from `registry.k8s.io`.

The image was successfully downloaded using:

```powershell
docker pull registry.k8s.io/ingress-nginx/controller:v1.12.1
```

After recreating the controller Pod, it successfully reached:

```text
1/1 Running
```

### Evidence 5 — Ingress Controller

**Insert Screenshot Here**

```powershell
kubectl get pods -n ingress-nginx

kubectl get ingressclass
```

**Figure 5: NGINX Ingress Controller and IngressClass**

---

## 7. Ingress Routing

The `frontend-ingress` resource was configured using the `nginx`
IngressClass.

The final routing configuration was:

| Path   | Service      | Port |
| ------ | ------------ | ---: |
| `/`    | frontend-svc | 8080 |
| `/api` | backend-svc  | 8080 |

This allows both the frontend and backend API to be accessed through the
same HTTP entry point.

### Evidence 6 — Ingress Routing

**Insert Screenshot Here**

```powershell
kubectl describe ingress frontend-ingress -n dso202-assignment-01
```

**Figure 6: Ingress Routing Configuration**

The screenshot should clearly show:

```text
/api → backend-svc:8080
/    → frontend-svc:8080
```

---

## 8. Frontend Configuration

Initially, the frontend was configured with:

```text
http://backend-svc:8080
```

This caused the browser to display:

```text
backend unreachable
```

The reason was that `backend-svc` is a Kubernetes internal DNS name.
It can be accessed from inside the Kubernetes cluster, but the user's
browser cannot directly resolve it.

The frontend configuration was therefore changed to:

```text
http://localhost:8080
```

This allowed browser requests to go through the Ingress.

The final rendered `config.js` was verified using:

```powershell
curl.exe http://localhost:8088/config.js
```

### Evidence 7 — Frontend Configuration

**Insert Screenshot Here**

The screenshot should show:

```text
BACKEND_URL: "http://localhost:8080"
```

This configuration allowed the browser to access:

```text
http://localhost:8088/api/tasks
```

through the local Ingress.

---

## 9. Application Testing

### 9.1 Frontend Test

The Field Log application was successfully accessed through the local
Ingress/port-forward.

**Insert Browser Screenshot Here**

**Figure 8: Final Field Log Application**

The application should show:

```text
backend + db online
```

---

### 9.2 Backend API Test

The `/api/tasks` endpoint was tested using:

```powershell
curl.exe http://localhost:8088/api/tasks
```

The API successfully returned the stored task records.

### Evidence 8 — GET /api/tasks

**Insert Screenshot Here**

**Figure 9: Successful GET /api/tasks Request**

---

### 9.3 Database Connectivity Test

The backend health endpoint was tested using:

```powershell
curl.exe http://localhost:8088/api/status
```

The response was:

```json
{
  "status": "ok",
  "db": "connected"
}
```

This confirms that the backend was successfully connected to PostgreSQL.

### Evidence 9 — Database Health

**Insert Screenshot Here**

**Figure 10: Successful Database Connectivity Test**

---

## 10. Verification Summary

| Component              | Result       |
| ---------------------- | ------------ |
| Frontend Pod           | Running      |
| Backend Pod            | Running      |
| PostgreSQL Pod         | Running      |
| PostgreSQL StatefulSet | 1/1 Ready    |
| PVC                    | Bound        |
| Backend Service        | ClusterIP    |
| Frontend Service       | NodePort     |
| Database Service       | Headless     |
| Ingress Controller     | Running      |
| IngressClass           | nginx        |
| `/` route              | Frontend     |
| `/api` route           | Backend      |
| `/api/tasks`           | Successful   |
| `/api/status`          | DB connected |

---

## 11. Troubleshooting

### 11.1 Ingress Controller Scheduling

Initially, the Ingress controller could not be scheduled because the
required node label was missing.

The control-plane node was labelled:

```powershell
kubectl label node control-plane ingress-ready=true
```

After applying the label, the controller was successfully scheduled.

### 11.2 Ingress Controller Image Pull

The controller initially entered:

```text
ErrImagePull
ImagePullBackOff
```

The Kubernetes event showed a:

```text
TLS handshake timeout
```

The controller image was manually pulled using Docker:

```powershell
docker pull registry.k8s.io/ingress-nginx/controller:v1.12.1
```

After the Pod was recreated, it reached:

```text
1/1 Running
```

### 11.3 Frontend Backend URL

The frontend initially showed:

```text
backend unreachable
```

The problem was caused by the browser trying to access:

```text
http://backend-svc:8080
```

This is an internal Kubernetes Service address.

The frontend was changed to use:

```text
http://localhost:8080
```

through the Ingress.

After recreating the frontend Pod, the application successfully
communicated with the backend.

### 11.4 Port Conflict

Port `8080` was already unavailable for local port forwarding.

Therefore, port `8088` was used for testing:

```text
http://localhost:8088
```

---

## 12. Conclusion

Assignment 2 was successfully completed.

The final Kubernetes environment contains the frontend, backend and
PostgreSQL database components. PostgreSQL is deployed using a StatefulSet
with persistent storage. Kubernetes Services provide communication between
the application components, while NGINX Ingress provides a single HTTP
entry point for frontend and API traffic.

The final tests confirmed that:

* The frontend application is accessible.
* The backend API is accessible.
* `/api/tasks` successfully returns task data.
* `/api/status` reports that the database is connected.
* PostgreSQL StatefulSet is running.
* The database PVC is bound.
* NGINX Ingress controller is running.
* `/` routes to the frontend.
* `/api` routes to the backend.

Therefore, the required Kubernetes deployment and Ingress configuration
were successfully implemented and verified.

---

# Screenshot Placement Checklist

Before submitting, insert the actual screenshots in this order:

1. **Figure 1:** Final Pods, Services, PVC and Ingress
2. **Figure 2:** PostgreSQL StatefulSet
3. **Figure 3:** PostgreSQL PVC
4. **Figure 4:** Backend Service and endpoint
5. **Figure 5:** NGINX controller and IngressClass
6. **Figure 6:** Ingress routing
7. **Figure 7:** Frontend `config.js`
8. **Figure 8:** Final Field Log application
9. **Figure 9:** `GET /api/tasks`
10. **Figure 10:** `GET /api/status`

```

This matches the report already found in your Library, but the version above is cleaned up around **your actual commands and final state**, including the `localhost:8088` testing and the `http://localhost:8080` frontend configuration. :contentReference[oaicite:2]{index=2}

**Important:** don't use the earlier `BACKEND_URL: "http://backend-svc:8080"` as your final frontend evidence. Your final working configuration is `http://localhost:8080`, and your API tests through `localhost:8088` succeeded.
```
