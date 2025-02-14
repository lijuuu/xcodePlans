### **🔥 Architecture Breakdown (Optimized for Latency & Scalability)**  

✅ **Send HTTP request with `user_id` as `wss_session_id`**  
✅ **NATS pub/sub for `code.run` (session-based execution requests)**  
✅ **Worker Pool with concurrency control (limits workers per t3.micro)**  
✅ **Prewarmed Firecracker VMs for ultra-low latency execution**  
✅ **Dynamic VM health checks & replacement strategy**  
✅ **Efficient WebSocket routing via session ID**  

---

### **🚀 Execution Flow (Step-by-Step)**
1️⃣ **Frontend sends execution request via HTTP**
   - `{ user_id, session_id, code, lang, ... }`
   - Backend validates, pushes to **NATS (`code.run`)**  

2️⃣ **Code Execution Service subscribes to `code.run`**
   - Dequeues request
   - Calls `getFreeVM()` to assign a worker  
   - **Worker Pool limits concurrency** (e.g., max 5 workers per t3.micro)  

3️⃣ **Firecracker VM executes code**
   - If VM is broken → **replace it automatically**  
   - If no free VM → **queue request (backpressure)**  

4️⃣ **Worker sends output to WebSocket via session ID**
   - Backend sends `{ session_id, output, status }`  

5️⃣ **Frontend WebSocket receives result in real-time**
   - UI updates instantly  

---

### **🔥 Optimizations for Scaling**
🔹 **Limit worker pool concurrency** → Prevents t3.micro overload  
🔹 **Prewarm Firecracker VMs** → <10ms startup time  
🔹 **Backpressure Handling** → Queue tasks if workers are full  
🔹 **Replace dead VMs dynamically** → Ensures execution continuity  
🔹 **NATS for async messaging** → Low-latency task distribution  
🔹 **WSS session ID routing** → Avoids unnecessary open connections  

---

### **🛠 Potential Issues & Fixes**
| **Issue**                 | **Fix**  |
|---------------------------|---------|
| Too many WebSocket connections? | Scale horizontally with **NATS JetStream + multiple backend instances** |
| Firecracker VMs crashing? | **Health check & auto-replace** broken VMs |
| Worker Pool saturation? | **Queue & rate-limit requests** |
| High latency on cold start? | **Prewarm VMs** and keep idle ones ready |
| User disconnects mid-execution? | **Store result for reconnection** or retry |
| t3.micro bottleneck? | **Upgrade instances or load balance execution** |

---

### **🔥 Performance Estimations on t3.micro**
| **Component** | **Estimated Latency (ms)** |
|--------------|--------------------------|
| HTTP Request to Backend | 1-5 ms |
| NATS Publish to Worker | 1-3 ms |
| Worker Dequeuing Task | 1-5 ms |
| Firecracker Startup (Prewarmed) | <10 ms |
| Code Execution (Simple) | 20-50 ms |
| Code Execution (Heavy) | 100-500 ms |
| NATS Publish Result | 1-3 ms |
| WebSocket Transmission | 1-5 ms |
| **Total (Prewarmed Firecracker, Simple Code)** | **30-70 ms** |
| **Total (Cold Firecracker, Heavy Code)** | **200-800 ms** |

---

### **🚀 Scaling Beyond t3.micro**
For **1M users, 10K concurrent requests**, you'd need:
- **Load balancing** (ALB, Nginx)
- **More instances (t3.large, c5.large)**
- **Distributed worker pool**
- **NATS JetStream for fault tolerance**

---

### **✅ Final Verdict**
This setup is **fast, scalable, and efficient** 🚀  
Want to **see sample code for this?** 🔥
