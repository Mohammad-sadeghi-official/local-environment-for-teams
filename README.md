# 🚀 Local Environment for Teams  
A realistic, production-like DevOps environment built using **Docker**, **Nginx**, **Flask**, **Redis**, **SELinux**, and **Firewalld** on **Fedora 41**.

This project simulates a real-world local environment for development teams.  
It includes a reverse proxy, isolated networks, backend API, frontend static hosting, caching system, and strict Linux security (SELinux + Firewalld enabled).

Perfect for DevOps learning, debugging, testing microservices, or onboarding new developers.

---

# 📦 **Architecture Overview**


### ✔ Services
| Service | Description |
|---------|------------|
| **Nginx** | Reverse proxy handling `/` and `/api/` routes |
| **Frontend** | Static HTML served via Nginx (alpine image) |
| **Backend** | Flask app running on port 5000 |
| **Redis** | Internal caching/message broker |
| **Docker Compose** | Orchestration of all services |
| **SELinux** | Enforced mode with custom policies |
| **Firewalld** | Port & network rules fully enabled |

---

# 🐳 **docker-compose.yml**

This environment uses **two Docker networks**:

- `web-net` → public traffic (Nginx + Frontend + Backend)
- `internal-net` → isolated traffic (Backend + Redis)

### Highlighted compose sections:
```yaml
nginx:
  image: nginx:latest
  volumes:
    - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:ro
  ports:
    - "80:80"
  networks:
    - web-net

backend:
  build: ./backend
  command: python app.py
  networks:
    - web-net
    - internal-net

redis:
  image: redis:alpine
  networks:
    - internal-net
```
گگگک'
compose file is located at:
./docker-compose.yml

nginx/default.conf
Nginx handles both frontend and backend routing:

✔ Frontend (/)
✔ Backend API (/api/)
✔ WebSocket support
✔ Proper Forwarded Headers
✔ Custom error page
✔ Full logging

server {
    listen 80;
    server_name myapp.local;

    location / {
        proxy_pass http://frontend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Upgrade $http_upgrade;
    }

    location /api/ {
        proxy_pass http://backend:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log warn;
}

Backend (Flask) : 
```
  from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/message')
def message():
    return jsonify({"message": "Hello from Backend!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Frontend (Static HTML):
```
<h1>Frontend Running ✅</h1>
<p id="msg">Loading...</p>

<script>
  fetch("/api/message")
    .then(res => res.json())
    .then(data => {
      document.getElementById("msg").innerText = data.message;
    });
</script>
```

🛠 Firebase Dockerfiles :
``
      Backend Dockerfile: 
     
          FROM python:3.11-slim
          WORKDIR /app
          COPY app.py .
          RUN pip install flask
          EXPOSE 5000
          CMD ["python", "app.py"]
``
        Frontend Dockerfile:
        
          FROM nginx:alpine
          COPY index.html /usr/share/nginx/html/index.html
          EXPOSE 80



  🔥 SELinux & Firewalld Configuration
✔ Allow Nginx & containers to connect to the backend

    sudo setsebool -P httpd_can_network_connect 1
    sudo setsebool -P container_connect_any 1

✔ Fix file context for mounted configs:

    sudo chcon -Rt container_file_t ./nginx
    
✔ Check SELinux denials:

    sudo ausearch -m AVC -ts recent
    
✔ Build custom SELinux policy module:

    sudo ausearch -m AVC -ts recent | audit2allow -M mynginx
    sudo semodule -i mynginx.pp
    
✔ Firewalld rules:

    sudo firewall-cmd --permanent --add-port=80/tcp
    sudo firewall-cmd --reload


🧾 Nginx Error Logs (Real Troubleshooting)
Throughout development, several real-world errors occurred:

❌ Permission Denied (SELinux)

      open() "/etc/nginx/conf.d/default.conf" failed (13: Permission denied)
      
❌ Upstream backend not ready

      connect() failed (111: Connection refused)
❌ Wrong service name in upstream

      host not found in upstream "back-end:5000"
      
❌ Frontend crash (Exit 127)

      /docker-entrypoint.sh: exec: line 47: npm: not found
      
These were solved by:

Fixing SELinux labels

Correcting compose networking

Adjusting nginx routes

Proper container build flow

Ensuring backend readiness


🚀 How to Run
docker compose up -d --build


Then open:

http://localhost


Or if using /etc/hosts:

http://myapp.local


⭐ Features
✔ Production-like architecture

✔ Reverse proxy with clean routing

✔ Multiple networks (isolated traffic)

✔ Backend API returning JSON

✔ Static frontend with API call

✔ Redis integration

✔ SELinux enforced

✔ Firewalld enabled

✔ Troubleshooting included

✔ Developer-friendly local environment

🧱 Future Improvements
HTTPS termination (Let's Encrypt / self-signed)

Docker secrets + environment files

CI/CD pipeline (GitHub Actions)

Container healthchecks

Logging driver improvements

Kubernetes version (minikube)

Monitoring (Prometheus + Grafana)

🙌 Contributions & Feedback
If you have experience with:

Nginx

SELinux

Docker networking

Reverse proxy

Local DevOps environments

I'd love to hear your feedback or suggestions.
















      
