# 🔧 Full CI/CD Pipeline with Jenkins | Node.js + React Deployment

This project demonstrates a complete CI/CD pipeline using **Jenkins**, with a backend in **NestJS (Node.js)** and frontend in **React**, deployed automatically with **build**, **rollback**, and **monitoring** features using **Prometheus-Grafana**.

---

## 🚀 Features

- ✅ Jenkinsfile for full CI/CD
- ✅ GitHub to Jenkins webhook integration
- ✅ Node.js + React automated build
- ✅ Rollback on failure
- ✅ Dockerized architecture
- ✅ SSH key setup for remote deployment
- ✅ Monitoring via Prometheus + Grafana

---

## 📁 Project Structure

├── backend/ # NestJS App
├── frontend/ # React App
├── Jenkinsfile # Jenkins pipeline script
├── docker-compose.yml # Multi-container orchestration



---

## 🛠️ Jenkins Pipeline Stages

1. **Clone GitHub Repo**
2. **Install Dependencies (`npm ci`)**
3. **Run Unit Tests**
4. **Build Docker Images**
5. **Deploy to Remote Server via SSH**
6. **Health Check**
7. **Rollback if Deployment Fails**
8. **Monitoring with Prometheus-Grafana**

---

## 📊 Monitoring Dashboard

- 📡 Prometheus collects metrics from backend & frontend containers
- 📊 Grafana shows live system status
- ⚠️ Alerts can be configured for CPU, memory, downtime, etc.

---

## 🔐 SSH Setup (For Deployment)

- Jenkins private/public key generated inside `~/.ssh`
- Public key copied to remote server using:
  ```bash
  ssh-copy-id jenkins@<remote-ip>


