# 🎥 Application Live Demo

---

This section showcases the operational validation of the VProfile application after deployment. Each screenshot demonstrates successful functionality across different components.

---

### 1️⃣ Application Login Screen

![application_login](../resources/Images/application_login.png)

Shows the login interface confirming that the application is reachable through the Load Balancer URL.

---

### 2️⃣ SSL Certificate Validation

![application_ssl](../resources/Images/application_ssl.png)

Demonstrates a valid HTTPS connection secured via AWS ACM certificate.

---

### 3️⃣ Application Home Screen

![application_homepage](../resources/Images/application_homepage.png)

Confirms successful authentication and UI functionality.

---

### 4️⃣ Database Connection Verification

![application_database](../resources/Images/application_database.png)

Shows the application successfully connecting to the MySQL backend using Route53 private DNS.

---

### 5️⃣ Memcached Connection Verification

![application_memcache](../resources/Images/application_memcache.png)

Validates that the caching layer (`mc01`) is reachable and functioning.

---

### 6️⃣ RabbitMQ Connection Verification

![application_rabbitmq](../resources/Images/application_rabbitmq.png)

Confirms messaging functionality between the application and RabbitMQ (`rmq01`).

---
