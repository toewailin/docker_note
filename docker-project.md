### **Docker Setup for Node.js Development Application**  
This documentation provides the complete setup for running a **Node.js** application in a **Docker container**. It walks through the process of creating a simple **Node.js application** using **Express**, configuring the application to run with **Docker**, and using **`nodemon`** for development. This guide is aimed at ensuring consistency across development environments and allows real-time code updates inside the container.

---

### **Project Structure**

The project structure for this application looks as follows:

```
/my-node-app
  ├── Dockerfile
  ├── package.json
  ├── server.js
  └── .dockerignore
```

### **1. Application Code (`server.js`)**

The core of the application is a basic **Express** app that listens on port `80` and serves two routes:

```js
// server.js
const express = require('express');
const app = express();
const port = 80;

// Basic route
app.get('/', (req, res) => {
  res.send('Hello World!!!!');
});

// About route
app.get('/about', (req, res) => {
  res.send('About');
});

// Start server
app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
});
```

### **2. `package.json` Setup**

The `package.json` file contains metadata about the project, including dependencies and scripts.

```json
{
  "name": "dnt",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",          // Command to run in production
    "dev": "nodemon server.js",         // Command for development with hot-reloading
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "express": "^4.21.2"                // Main application dependency (Express)
  },
  "devDependencies": {
    "nodemon": "^2.0.15"                // Development dependency for auto-reload
  }
}
```

### **3. Dockerfile Configuration**

This `Dockerfile` defines the environment and how to build the Docker image for the Node.js application.

```dockerfile
# Use Node.js Alpine image for a smaller footprint
FROM node:23-alpine3.20

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json /app
RUN npm install

# Install nodemon globally for development (for auto-reloading)
RUN npm install -g nodemon

# Copy the rest of the application code into the container
COPY . /app

# Expose port 80 (to be mapped to the host's port 3000)
EXPOSE 80

# Command to start the app using nodemon for development mode
CMD ["npm", "run", "dev"]
```

- **`FROM node:23-alpine3.20`**: This uses the official Node.js Alpine image, which is lightweight.
- **`WORKDIR /app`**: Sets the working directory inside the container to `/app`.
- **`COPY package.json /app`**: Copies `package.json` into the container.
- **`RUN npm install`**: Installs the application dependencies listed in `package.json`.
- **`RUN npm install -g nodemon`**: Installs `nodemon` globally for automatic server restarts during development.
- **`COPY . /app`**: Copies the application code into the container.
- **`EXPOSE 80`**: Exposes port 80 in the container (mapped to port 3000 on the host).
- **`CMD ["npm", "run", "dev"]`**: Uses `nodemon` to run the application in development mode (auto-reloading).

### **4. Creating `.dockerignore`**

The `.dockerignore` file ensures that unnecessary files (like `node_modules`) are not included in the Docker image.

Create a `.dockerignore` file with the following content:

```
node_modules
```

This prevents the local `node_modules` from being copied into the container, ensuring that the container installs its own dependencies.

### **5. Building the Docker Image**

To build the Docker image from the `Dockerfile`, run the following command in your terminal:

```bash
docker build -t my-node-app .
```

- **`-t my-node-app`**: This tag names the image `my-node-app`.
- **`.`**: The dot specifies the current directory as the build context.

This will create a Docker image that contains your Node.js application along with all the necessary dependencies.

### **6. Running the Docker Container**

To run the container with your application, use the following command:

```bash
docker run -it -p 3000:80 -v $(pwd):/app --name my-node-app-container my-node-app
```

#### Explanation of the command:
- **`-it`**: Runs the container in interactive mode, allowing you to interact with the container if necessary.
- **`-p 3000:80`**: Maps port `80` inside the container to port `3000` on your host machine. You can access your application via `http://localhost:3000`.
- **`-v $(pwd):/app`**: Mounts the current directory (`$(pwd)`) from your host machine to the `/app` directory inside the container. This allows live updates to the application code.
- **`--name my-node-app-container`**: Names the running container `my-node-app-container`.
- **`my-node-app`**: Specifies the Docker image to run (which you built earlier).

### **7. Accessing the Application**

After running the Docker container, you can access your application in the browser at:

```
http://localhost:3000
```

You should see the following output in your terminal:

```
[nodemon] 3.1.9
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,cjs,json
[nodemon] starting `node server.js`
Example app listening on port 80
```

Now, your application should be running, and you can access the two routes defined in `server.js`:
- `http://localhost:3000/` → "Hello World!!!!"
- `http://localhost:3000/about` → "About"

### **8. Editing Code and Live Refresh**

Since you mounted your local directory to `/app` in the container using the `-v` flag, any changes you make locally to your files will be immediately reflected inside the running container. **`nodemon`** will automatically detect these changes and restart the application.

For example:
- Edit `server.js` locally.
- Save the file, and `nodemon` will automatically restart the server.

### **9. Stopping and Removing the Container**

Once you're done working with the container, you can stop it using the following command:

```bash
docker stop my-node-app-container
```

If you want to remove the container completely, run:

```bash
docker rm my-node-app-container
```

### **Conclusion**

This guide provides a full workflow for running a **Node.js** application inside a **Docker container** with **Express** and **`nodemon`** for development. Using Docker for development allows you to have a consistent environment across machines, ensuring that "it works on my machine" issues are minimized.

Key Benefits:
- **Consistency**: The same environment on every machine.
- **Development Efficiency**: Real-time code updates with `nodemon`.
- **Isolation**: The application and its dependencies are isolated from the host system.

By following this process, you can now effectively develop and test your Node.js applications in a Docker container. Let me know if you need further details or assistance!
