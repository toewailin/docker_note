# Stop and Remove All Containers**

### **List of Commands:**

1. **Stop and remove containers**:
   ```bash
   docker compose down
   ```

2. **Remove all containers** (including stopped ones):
   ```bash
   docker rm -f $(docker ps -a -q)
   ```

3. **Remove all images**:
   ```bash
   docker rmi -f $(docker images -q)
   ```

4. **Remove unused volumes**:
   ```bash
   docker volume prune -f
   ```

5. **Remove unused networks**:
   ```bash
   docker network prune -f
   ```

6. **Remove Docker build cache**:
   ```bash
   docker builder prune -a -f
   ```

7. **Remove everything (optional)**:
   ```bash
   docker system prune -a -f --volumes
   ```
