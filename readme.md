# Dapr Sidecar Implementation — .NET Example

Using **Redis Pub/Sub** for event-based communication.

---

## 🐾 Overview

This example demonstrates how **Dapr sidecars** enable communication between services using Redis Pub/Sub.

* **Pet Service** → publishes events
* **Rescue Service** → subscribes to events
* **Hospital Service** → subscribes to events

---

## 🔧 Architecture

```
Pet Service (Publisher)
        |
        | publishes events → pubsub (Redis)
        v
+------------------+
|   Dapr Sidecar   |
|       (PET)      |
+------------------+
        |
        v
Redis Pub/Sub  <---------------------------------------------+
(pet-flagged-for-adoption, pet-transferred-to-hospital topics) |
        |                                                      |
        |                                                      |
        v                                                      |
+------------------+          +--------------------+          |
| Dapr Sidecar     |          | Dapr Sidecar       |          |
|   (RESCUE)       |          |   (HOSPITAL)       |
|   Subscriber     |          |   Subscriber       |
+------------------+          +--------------------+          |
        |                                                      |
        +------------------------------------------------------+
```

---

## ✅ Required Setup

### 1. Run Redis Commander (UI for Redis monitoring)

```sh
& "$(npm config get prefix)\redis-commander.cmd" --port 8082
```

✅ Opens UI at: [http://localhost:8082](http://localhost:8082)

---

### 2. Navigate to each service

Example:

```sh
cd src/WisdomPetMedicine.Hospital.Api
```

---

### 3. Run each service with its own sidecar

```sh
dapr run --app-id pet --app-port 5000 --dapr-http-port 50000 --components-path ../../components -- dotnet run --urls http://+:5000

dapr run --app-id rescue --app-port 5001 --dapr-http-port 50001 --components-path ../../components -- dotnet run --urls http://+:5001

dapr run --app-id hospital --app-port 5002 --dapr-http-port 50002 --components-path ../../components -- dotnet run --urls http://+:5002
```

---

## 🧠 Optional: Monitor Redis Pub/Sub

Open Redis CLI:

```sh
redis-cli
```

Monitor events:

```sh
MONITOR
```

---

## 🌐 Test API (Swagger)

Pet API Swagger UI:
👉 [http://localhost:5000/swagger](http://localhost:5000/swagger)

From Swagger you can:

* Add a pet
* Flag it for adoption → publishes event
* Transfer pet to hospital → publishes another event

---

## ⚙️ pubsub.yaml (Dapr Redis Component)

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.redis
  version: v1
  metadata:
    - name: redisHost
      value: "localhost:6379"
```

📝 **Important:**
`metadata.name` (here **pubsub**) must match the name used in your .NET code:

```csharp
public class PetApplicationService
private const string PubSubName = "pubsub";
```

## ✅ Repository Structure (from your /src directory)


---txt

src/
├── WisdomPetMedicine.Common               <-- Shared logic (DTOs / constants / events)
├── WisdomPetMedicine.Pet.Api              <-- Microservice: Pet (Publisher)
├── WisdomPetMedicine.Pet.Domain           <-- Domain layer for Pet Service
├── WisdomPetMedicine.Rescue.Api           <-- Microservice: Rescue (Subscriber / Publisher)
├── WisdomPetMedicine.Rescue.Domain        <-- Domain layer for Rescue Service
├── WisdomPetMedicine.RescueQuery.Api      <-- Read-model projection for Rescue service
├── WisdomPetMedicine.Hospital.Api         <-- Microservice: Hospital (Subscriber)
├── WisdomPetMedicine.Hospital.Domain      <-- Domain layer for Hospital service
├── WisdomPetMedicine.Hospital.Infrastructure <-- DB layer for hospital service
└── WisdomPetMedicine.sln

---

🔥 Done! Run the services and test from Swagger!

---





