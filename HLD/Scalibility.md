# Scalibility

## 1. Vertical Scaling VS Horizontal Scaling

### ⬆️ **Vertical Scaling**

- referred to as **"scale up"**, mean adding more power (CPU, RAM, etc.) to servers
- Particularly handy, especially because of its simplicity
- But comes with major drawbacks:
  -- Has a hard limit: impossible to add unlimited cpus, ram to a single server
  -- Does not come with failover and redundancy: one fails, system goes down

### ↔️ **Horizontal Scaling**

- referred to as **"scale-out"**, allows us to scale by adding more servers
- more desireable for large applications due to limitations of vertical scaling

---

![Vertical vs. Horizontal Scaling](../images/verticalVsHorizontalScaling.png)

## 2. Load Balancing

- A load balancer is a hardware appliance, a software service, or a cloud resource that distributes incoming network traffic across a group of backend servers (a “server farm”). Its job is to ensure that no single server gets overwhelmed, which improves availability, responsiveness, and scalability.

- In a horizontally scaled architecture, the load balancer is the entry point. It lets us add (or remove) server instances at will – the balancer automatically discovers them and begins distributing traffic, giving us elastic scalability.

### 2.1 Load‑Balancing Algorithms (Rules)

The algorithm is the rule that decides which backend server gets each new request. This is the core of load‑balancer configuration, and the Pareto principle applies strongly: three or four algorithms cover 80%+ of real‑world scenarios.

#### 2.1.1 Round‑Robin (and Weighted Round‑Robin)

- **How it works**: The load balancer maintains a list of servers and sends each new request to the next server in the list, cycling back to the start after the last one. **Weighted Round‑Robin** assigns a “weight” to each server based on its capacity (e.g., a server with 2× the CPU gets weight 2). The balancer then distributes requests proportionally.

- **When to use**: Stateless applications with servers of equal (or known‑different) horsepower, where requests are short‑lived and roughly uniform. Classic example: a cluster of identical web servers serving static content.

- **Limitation**: Doesn’t consider current server load. If one request is unusually slow, that server may become overloaded while others sit idle.
