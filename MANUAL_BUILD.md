# Manual Build Instructions for MERN Application

This guide provides step-by-step instructions to build and run the MERN (MongoDB, Express, React, Node.js) application manually on your local machine before using Docker Compose for automation.

## Prerequisites

Ensure you have the following installed on your system:
- **Node.js** (v18.x or later recommended)
- **npm** (comes with Node.js)
- **MongoDB Community Server** (for local runs)
- **Docker Desktop** (for containerization)
- **Docker Hub Account** (for pushing images)

---

## Part 1: Local Manual Build (No Docker)

### Step 1: Set Up the Database
1.  **Start MongoDB**: Ensure your local MongoDB service is running. 
    - Default connection string: `mongodb://localhost:27017`
2.  **Create Database**: The application uses a database named `employees`.

### Step 2: Build and Run the Backend
1.  **Navigate to the backend directory**: `cd mern/backend`
2.  **Install Dependencies**: `npm install`
3.  **Configure Database Connection**: Change the URI in `mern/backend/db/connection.js` to `mongodb://localhost:27017`.
4.  **Start the Server**: `npm start` (Runs on port 5050)

### Step 3: Build and Run the Frontend
1.  **Navigate to the frontend directory**: `cd mern/frontend`
2.  **Install Dependencies**: `npm install`
3.  **Start the Development Server**: `npm run dev` (Runs on port 5173)

---

## Part 2: Building with Dockerfiles (Manual Containerization)

In this section, we build and run each tier manually using `docker` commands without `docker-compose`.

### Step 1: Create a Docker Network
Create a bridge network so the containers can communicate using their names:
```bash
docker network create mern-network
```

### Step 2: Run MongoDB Container
```bash
docker run -d \
  --name mongodb \
  --network mern-network \
  -p 27017:27017 \
  -v mongo-data:/data/db \
  mongo:latest
```

### Step 3: Build and Run Backend Image
1.  **Build the image**:
    ```bash
    cd mern/backend
    docker build -t mern-backend:v1 .
    ```
2.  **Run the container**:
    ```bash
    docker run -d \
      --name mern-backend \
      --network mern-network \
      -e MONGO_URI=mongodb://mongodb:27017/employees \
      -p 5050:5050 \
      mern-backend:v1
    ```

### Step 4: Build and Run Frontend Image
1.  **Build the image**:
    ```bash
    cd ../frontend
    docker build -t mern-frontend:v1 .
    ```
2.  **Run the container**:
    ```bash
    docker run -d \
      --name mern-frontend \
      --network mern-network \
      -p 5173:5173 \
      mern-frontend:v1
    ```

---

## Part 3: Pushing to Docker Hub

To share your images publicly, follow these steps to push them to Docker Hub.

### Step 1: Login to Docker Hub
```bash
docker login
```
*Enter your Docker Hub username and password when prompted.*

### Step 2: Tag Your Images
Replace `<your-username>` with your actual Docker Hub username.
```bash
# Tag Backend
docker tag mern-backend:v1 <your-username>/mern-backend:latest

# Tag Frontend
docker tag mern-frontend:v1 <your-username>/mern-frontend:latest
```

### Step 3: Push Images to Docker Hub
```bash
# Push Backend
docker push <your-username>/mern-backend:latest

# Push Frontend
docker push <your-username>/mern-frontend:latest
```

### Step 4: Verify on Docker Hub
1.  Go to [hub.docker.com](https://hub.docker.com/).
2.  Check your repositories; you should see `mern-backend` and `mern-frontend` listed.

---

## Part 4: Automated CI/CD with GitHub Actions

The manual steps above are now automated using GitHub Actions.

### Automated Workflow
1.  **Continuous Integration**: Every push to `main` or `develop` triggers linting and build tests.
2.  **Continuous Deployment**: 
    - Pushes to `develop` automatically build and deploy to the **Staging** environment.
    - Pushes to `main` automatically build and deploy to the **Production** environment.
3.  **Docker Hub Integration**: Images are automatically built, tagged with the commit SHA, and pushed to Docker Hub.
4.  **Notifications**: Success or failure alerts are sent to Slack.

### Setup for Automation
1.  Fork this repository.
2.  Add the required secrets (DOCKERHUB_USERNAME, DOCKERHUB_TOKEN, EC2_HOST, EC2_USERNAME, EC2_SSH_KEY, SLACK_WEBHOOK) to your GitHub repository settings.
3.  The pipeline will handle the rest!

---

## Summary of Ports
- **Frontend**: 5173
- **Backend**: 5050
- **MongoDB**: 27017
