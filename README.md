# CST8915 Lab 2: 12-Factor Refactor of the Algonquin Pet Store (4 Azure VMs)

**Student Name:** Georges Ngepi
**Student ID:** 041131489
**Course:** CST8915 Full-stack Cloud-native Development
**Semester:** Fall 2026

---

## Demo Video

🎥 Video link: coming (will be added here)

---

## Service Repositories

| Service | Technology | Repository |
|---|---|---|
| Order Service | Node.js / Express | https://github.com/NgepiGeorges/order-service |
| Product Service | Rust / Warp | https://github.com/NgepiGeorges/product-service |
| Store Front | Vue.js | https://github.com/NgepiGeorges/store-front |

## Deployment (4 Azure VMs)

| VM | Runs | Port | Region | Inbound rule (source) |
|---|---|---|---|---|
| rabbitmq-vm | RabbitMQ broker | 5672 | Sweden Central | order-vm public IP only |
| product-vm | Product Service | 3030 | Sweden Central | my IP only |
| order-vm | Order Service | 3000 | Belgium Central | my IP only |
| store-vm | Store Front | 8080 | Belgium Central | my IP only |

---

## Reflection Questions

### 1. What changes did I make to comply with the Configuration and Backing Services factors?

In Lab 1, the RabbitMQ address (`amqp://localhost`) and the ports were written directly in the code. In the `order-service`, I now read `RABBITMQ_CONNECTION_STRING` and `PORT` from environment variables, loaded from a local `.env` file with the `dotenv` package. In the `product-service`, I added the `dotenv` crate and read `PORT` from the environment (default 3030). The real `.env` files are in `.gitignore`, and each repository has a `.env.example` with placeholders only. For Backing Services, RabbitMQ is now an attached resource on its own VM: the Order Service reaches it only through the connection URL, with a dedicated `orderapp` account. To move to another broker, I only change the URL, not the code.

### 2. Why use environment variables instead of hard-coded configuration?

Environment variables let the same code run in different places (my laptop, a VM, a container) without editing the source. In this lab, the RabbitMQ VM had a different IP than in Lab 1, and I did not need to change any code, only the `.env` file. They also keep secrets like the RabbitMQ password out of Git, so they are never published on GitHub. Finally, configuration can be changed without rebuilding or redeploying the code.

### 3. Why have separate repositories for each microservice?

Each service has its own repository, its own dependencies (`package-lock.json` or `Cargo.lock`) and its own history. This means one team can change, test and deploy the Order Service without touching the Product Service or the Store Front. It also allows each service to use the best language for its job (Node.js, Rust, Vue). For scalability, each service can be deployed and scaled on its own: in this lab each one runs on its own VM, so I could give more resources to one service without changing the others.

---

## Challenges and Lessons Learned

- **Azure for Students quotas.** The subscription allows only 6 vCPUs and 3 public IPs per region. Stopped VMs still count toward the vCPU quota, so I deleted my Lab 1 resource group. I placed two VMs in Sweden Central and two in Belgium Central, because Canada Central is blocked by my subscription policy.
- **My public IP changes.** My IP at school is different from my IP at home, so the "my IP only" NSG rules stopped working when I moved. I had to update the rule source.
- **Services stop when SSH disconnects.** My SSH connection dropped several times, and the service stopped with it. I used `nohup ... &` to run the services in the background.
- **Least privilege.** Port 5672 accepts traffic only from the order-vm IP, and the `orderapp` RabbitMQ user has no administrator tag.
