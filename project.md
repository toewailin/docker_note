### **Project Structure**

Here’s the structure of the project:

```
my-app/
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── (other Node.js backend files)
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   └── (other React frontend files)
│
├── nginx/
│   ├── default.conf (Nginx reverse proxy config)
│
└── docker-compose.yml
```

### **1. Backend (Node.js)**

In your **backend** folder, create a `Dockerfile` to containerize the Node.js app.

#### **backend/Dockerfile**

```Dockerfile
# Use an official Node.js runtime as a parent image
FROM node:16

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of your application code
COPY . .

# Expose the port that your app runs on
EXPOSE 5000

# Start the app
CMD ["npm", "start"]
```

Make sure your Node.js application is listening on port `5000` (or any port you prefer). Modify the port in the `Dockerfile` and application code as needed.

### **2. Frontend (React)**

In your **frontend** folder, create a `Dockerfile` to containerize the React app.

#### **frontend/Dockerfile**

```Dockerfile
# Use an official Node.js runtime as a parent image
FROM node:16 AS build

# Set the working directory in the container
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# Build the React app for production
RUN npm run build

# Use a lightweight web server to serve the static files
FROM nginx:alpine

# Copy the build output to Nginx's public directory
COPY --from=build /app/build /usr/share/nginx/html

# Expose port 80 for the frontend
EXPOSE 80

# Start Nginx server
CMD ["nginx", "-g", "daemon off;"]
```

This Dockerfile does two things:
1. It uses Node.js to build the React app in the first stage.
2. It uses a lightweight Nginx image to serve the built React app.

### **3. Nginx (Reverse Proxy)**

You’ll need an **Nginx** configuration file to reverse proxy requests from the frontend to the backend.

#### **nginx/default.conf**

```nginx
server {
  listen 80;

  # React frontend
  location / {
    root /usr/share/nginx/html;
    try_files $uri /index.html;
  }

  # API requests for backend (Node.js)
  location /api/ {
    proxy_pass http://backend:5000/; # This connects to the backend container
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```

In this configuration:
- **Frontend:** It serves the static React files from `/usr/share/nginx/html`.
- **Backend API:** It proxies requests to `/api/` to the Node.js backend, which will be running on port `5000`.

### **4. Docker Compose Configuration**

Now let’s define how all of these services (Node.js, React, PostgreSQL, Redis, and Nginx) will work together in a `docker-compose.yml` file.

#### **docker-compose.yml**

```yaml
version: '3.8'

services:
  # Node.js backend
  backend:
    build:
      context: ./backend
    container_name: backend
    environment:
      - NODE_ENV=production
      - DB_HOST=postgres
      - DB_USER=postgres
      - DB_PASSWORD=example
      - DB_NAME=mydb
      - REDIS_HOST=redis
    ports:
      - "5000:5000" # Map container port to host port
    depends_on:
      - postgres
      - redis
    networks:
      - app-network

  # React frontend
  frontend:
    build:
      context: ./frontend
    container_name: frontend
    ports:
      - "80:80"  # Expose the React app on port 80
    depends_on:
      - backend
    networks:
      - app-network

  # PostgreSQL database
  postgres:
    image: postgres:alpine
    container_name: postgres
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=example
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - app-network

  # Redis
  redis:
    image: redis:alpine
    container_name: redis
    networks:
      - app-network

  # Nginx reverse proxy
  nginx:
    build:
      context: ./nginx
    container_name: nginx
    ports:
      - "80:80"  # Expose Nginx on port 80
    depends_on:
      - frontend
      - backend
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  postgres_data:
```

### **Explanation of `docker-compose.yml`**
- **backend:** This service builds the Node.js backend, exposes port `5000`, and links it with PostgreSQL and Redis.
- **frontend:** This service builds the React app and serves it using Nginx.
- **postgres:** PostgreSQL database service, using `postgres:alpine` image.
- **redis:** Redis service, using `redis:alpine` image.
- **nginx:** Nginx reverse proxy that serves the React app and forwards API requests to the backend.
- **Networks:** All services are connected via the `app-network`, which is a bridge network for inter-service communication.

### **5. Running the Application**

To build and run your application with Docker Compose, follow these steps:

1. **Build the Docker images:**

   ```bash
   docker-compose build
   ```

2. **Start the containers:**

   ```bash
   docker-compose up
   ```

   This will start all services and show logs in the terminal.

3. **Access the application:**
   - Frontend (React app): [http://localhost](http://localhost)
   - Backend (API): `http://localhost:5000/api/` (for example, `/api/users`)

### **6. Stopping the Containers**

To stop the containers, you can use:

```bash
docker-compose down
```

This will stop and remove the containers, networks, and volumes.

---

### **Final Notes**

- Make sure your backend is correctly handling requests through `/api/` in the routes, and that it correctly interacts with the PostgreSQL database and Redis.
- You can scale the backend or frontend by adding more replicas if needed.
- For production use, remember to configure environment variables securely and set up proper database migrations for PostgreSQL.

By following these steps, you now have a professional Dockerized setup for your full-stack application! Feel free to ask if you need further clarification or if you run into any issues during setup.
