# django-todo
A simple todo app built with django

### **Step 1: Clone the code**
```
$ git clone https://github.com/Shubham230802/django-todo-K8s.git
```
### **Step 2: Install dependencies**
You will need django to be installed in you computer to run this app. Head over to https://www.djangoproject.com/download/ for the download guide

Once you have downloaded django, go to the cloned repo directory and run the following command

```bash
$ python manage.py makemigrations
```

This will create all the migrations file (database migrations) required to run this App.

Now, to apply this migrations run the following command
```bash
$ python manage.py migrate
```
following command and provide username, password and email for the admin user
```bash
$ python manage.py createsuperuser
```
 Start the server by following command

```bash
$ python manage.py runserver
```

Once the server is hosted, head over to http://127.0.0.1:8000/todos for the App.

## **Part 2: Dockerizing the Django application**

### **Step 1: Create a Dockerfile**
Create a **`Dockerfile`** in the root directory of the project with the following contents:

```
# Use the official Python image as the base image
FROM python:3
# Set the working directory in the container
WORKDIR /data
RUN pip install django==3.2
# Copy the application code to the working directory
COPY . .
RUN python manage.py migrate
# Expose the port on which the Django app will run
EXPOSE 8000
# Start the Flask app when the container is run
CMD ["python","manage.py","runserver","0.0.0.0:8000"]
```
### **Step 2: Build the Docker image**
```
docker build -t shubham2323/django-todo:latest .
```

### **Step 3: Run the Docker container**

To run the Docker container, execute the following command:

```
docker run -p -d 8000:8000 shubham2323/django-todo:latest
```
This will start the Django server in a Docker container on **`localhost:8000`**.

## **Part 3: Pushing the Docker image to Docker Hub**
```
docker push shubham2323/django-todo
```
## **Part 4: Manage docker container on k8s Pod **
```
#Use the same name as metadata and container
apiVersion: v1

kind: Pod

metadata:

  name: todo-pod

spec:

  containers:

  - name: todo-app
#Give image path as image within docker hub
image: shubham2323/django-todo:latest
#Start container on given port
ports:
    - containerPort: 8000
```
## **Part 5: Create Kubernetes deployment **
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: todo-deployment
  labels:
    app: todo-app
#In spec give the number of replicas that used a backup having the ability of auto-healing
spec:
  replicas: 3
  selector:
    matchLabels:
      app: todo-app
  template:
    metadata:
      labels:
        app: todo-app
    spec:
      containers:
      - name: todo-app
        image: shubham2323/django-todo:latest
#Container start at port 8000
        ports:
        - containerPort: 8000
```

## **Part 6: Create Kubernetes service **
for managed network and services with Host IP allocation through a proxy on AWS EC2 and Route53
```
apiVersion: v1
kind: Service
metadata:
  name: todo-service
spec:
  type: NodePort
  selector:
    app: todo-app
  ports:
      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
    - port: 80
      targetPort: 8000
      # Optional field
      # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
      nodePort: 30007
```

