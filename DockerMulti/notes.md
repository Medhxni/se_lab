Correct. If the exam asks for a **multi-container Docker application**, simply running an Nginx container and a MySQL container separately may not be enough if they expect communication between services. Let's make a proper example where the web application connects to MySQL.

We'll use **Node.js + Express + MySQL**, with Docker Compose managing both containers. The web application will actually query the database and display the result.

# **Experiment: Docker Multi-Container Application with Web + MySQL**

## **1. Folder structure**

Create this folder on GitHub:

```
DockerMulti/
├── Dockerfile
├── docker-compose.yml
├── package.json
└── index.js
```

## **2. Create&#x20;**`package.json`

Filename:

`DockerMulti/package.json`

```
{
  "name": "docker-multi-container",
  "version": "1.0.0",
  "description": "Web application connected to MySQL",
  "main": "index.js",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.21.2",
    "mysql2": "^3.14.0"
  }
}
```

This file specifies the Node.js dependencies required by the web application.

## **3. Create&#x20;**`index.js`

Filename:

`DockerMulti/index.js`

```
const express = require("express");
const mysql = require("mysql2");

const app = express();

const db = mysql.createConnection({
    host: "mysql",
    user: "root",
    password: "root123",
    database: "labdb"
});

app.get("/", (req, res) => {
    db.query("SELECT NOW() AS time", (err, result) => {
        if (err) {
            return res.send("Database connection error: " + err.message);
        }

        res.send(`
            <h1>Docker Multi-Container Application</h1>
            <h2>Web Container is Running!</h2>
            <p>MySQL Database Connected Successfully!</p>
            <p>Database Time: ${result[0].time}</p>
        `);
    });
});

app.listen(3000, () => {
    console.log("Web application running on port 3000");
});
```

### **Important concept**

```
host: "mysql"
```

Here `mysql` is the **Docker Compose service name** of the database container.

Docker Compose creates a network for the services, allowing the web container to reach the database using the hostname `mysql`. We don't need to know the database container's IP address.

## **4. Create&#x20;**`Dockerfile`

Filename:

`DockerMulti/Dockerfile`

```
FROM node:20

WORKDIR /app

COPY package.json .

RUN npm install

COPY index.js .

EXPOSE 3000

CMD ["npm", "start"]
```

### **What each instruction does**

| Instruction            | Purpose                                 |
| ---------------------- | --------------------------------------- |
| `FROM node:20`         | Uses Node.js image                      |
| `WORKDIR /app`         | Sets working directory inside container |
| `COPY package.json .`  | Copies dependency file                  |
| `RUN npm install`      | Installs Express and MySQL packages     |
| `COPY index.js .`      | Copies application code                 |
| `EXPOSE 3000`          | Documents application's port            |
| `CMD ["npm", "start"]` | Starts the web application              |

## **5. Create&#x20;**`docker-compose.yml`

Filename:

`DockerMulti/docker-compose.yml`

```
services:

  web:
    build: .
    container_name: webcontainer
    ports:
      - "8080:3000"
    depends_on:
      mysql:
        condition: service_healthy

  mysql:
    image: mysql:8
    container_name: dbcontainer
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: labdb
    ports:
      - "3307:3306"
    volumes:
      - dbdata:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uroot", "-proot123"]
      interval: 5s
      timeout: 5s
      retries: 10

volumes:
  dbdata:
```

### **Understand the ports**

## **Port mapping**

**Your browser**

`localhost:8080`

**Web container**

`3000`

**Web container**

`web`

**Database container**

`mysql:3306`

Inside Docker, the web service connects to MySQL using mysql:3306. Port 3307 is only for accessing MySQL from outside the Docker network, such as your host computer.

- `8080:3000` = host port 8080 → container port 3000.
- `3307:3306` = host port 3307 → container port 3306.
- `mysql` = service hostname used by the web app to reach the database.

The `depends_on` health condition makes Docker Compose wait until MySQL reports healthy before starting the web application.

# **6. Execution commands for your external exam**

Assume you create the project folder on the college PC and put all four files inside `DockerMulti`.

### **Step 1: Open terminal in project directory**

```
cd DockerMulti
```

### **Step 2: Build the web image and start both containers**

```
docker compose up -d --build
```

- `up` → creates and starts services.
- `-d` → runs in background.
- `--build` → builds the web image before starting.

Docker Compose automatically starts both the web service and the MySQL service.

### **Step 3: Check containers**

```
docker compose ps
```

Expected: both `web` and `mysql` services should be running, with the database eventually showing healthy.

Or:

```
docker ps
```

### **Step 4: Check logs**

```
docker compose logs
```

For only the web container:

```
docker compose logs web
```

For the database:

```
docker compose logs mysql
```

### **Step 5: Access the web application**

Open the browser and enter:

### [**http://localhost:8080**](http://localhost:8080?utm_source=chatgpt.com)

Open localhost:8080

Expected output:

# **Docker Multi-Container Application**

## **Web Container is Running!**

MySQL Database Connected Successfully!

Database Time: 2026-11-27 ...

The database time is returned from an actual SQL query:

```
SELECT NOW() AS time;
```

So this isn't merely two containers existing in the same vicinity. The web app actually sends a query to the MySQL database and receives a result.

# **7. Verify database separately (optional, but useful in viva)**

You can directly execute an SQL query inside the database container:

```
docker compose exec mysql mysql -uroot -proot123 -e "SHOW DATABASES;"
```

To check the database you created:

```
docker compose exec mysql mysql -uroot -proot123 labdb -e "SELECT NOW();"
```

To check running services:

```
docker compose ps
```

## **8. Stop and remove containers**

When the experiment is complete:

```
docker compose down
```

This stops and removes the containers and Compose network, but **preserves the database data volume**.

To delete the database volume as well, use:

```
docker compose down -v
```

Be careful: `-v` deletes the stored database data. The computer does not ask whether you've emotionally attached yourself to your data.

## **Complete command sequence to memorize**

### **Docker Multi-Container: Exam Cheat Sheet**

```
cd DockerMulti

docker compose up -d --build

docker compose ps

docker compose logs

docker compose exec mysql mysql -uroot -proot123 -e "SHOW DATABASES;"
```

Open in browser:

```
http://localhost:8080
```

Cleanup:

```
docker compose down
```

## **What to say during viva**

- **Why Docker Compose?** It manages multiple containers through a single YAML file.
- **Why two services?** One runs the web application, and another runs the MySQL database.
- **How does web connect to MySQL?** Using the service name `mysql` as the hostname on Docker's internal network.
- **Why not use localhost for MySQL inside the web container?** Because localhost refers to the web container itself, not the database container.
- **What is a volume?** It stores persistent database data outside the container's disposable filesystem.
- **Why&#x20;**`depends_on`**?** To control startup order and, with the health condition, wait for MySQL to become healthy.
- **What does&#x20;**`-d`**&#x20;do?** Runs containers in detached/background mode.
- **What does&#x20;**`--build`**&#x20;do?** Builds the web image using the Dockerfile.

This is a much better multi-container experiment to keep in your GitHub repo because it demonstrates **container creation, networking, service dependency, database connectivity, data persistence, logs, and verification** in one experiment.
