# tka-modul-5

# NGINC.CONF
```
events {}

http {
    upstream backend_roundrobin {
        server flask_app1:5000 weight=1;
        server flask_app2:5000 weight=1;
        server flask_app3:5000 weight=1;
        server flask_app4:5000 weight=1;
        server flask_app5:5000 weight=1;
    }
    upstream backend_weighted {
        server flask_app1:5000 weight=1;
        server flask_app2:5000 weight=2;
        server flask_app3:5000 weight=3;
        server flask_app4:5000 weight=4;
        server flask_app5:5000 weight=5;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_roundrobin;
        }
    }
}
```

# DOCKERFILE
```
# flask_app/Dockerfile
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

# REQUIREMENTS.TXT
```
Flask
pymongo

```
# APP.PY
```
from flask import Flask, jsonify
import time
from pymongo import MongoClient

app = Flask(__name__)

client = MongoClient('mongodb://mongodb1:27017,mongodb2:27017,mongodb3:27017,mongodb4:27017,mongodb5:27017')
db = client["MyDatabase_C07"]
collection = db["C_Tujuh"]

@app.route('/slow')
def slow():
    time.sleep(2)
    return jsonify({"message": "This is a slow response"})

@app.route('/fast')
def fast():
    time.sleep(0.5)
    return jsonify({"message": "This is a fast response"})

@app.route('/data')
def data():
    data = list(collection.find({}, {"_id": 0}))
    return jsonify(data)

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

# DOCKER-COMPOSE.YML
```
version: '3.8'

services:
  mongodb1:
    image: mongo
    container_name: mongodb1
    ports:
      - "27017:27017"
    networks:
      - mongo_network

  mongodb2:
    image: mongo
    container_name: mongodb2
    ports:
      - "27018:27017"
    networks:
      - mongo_network

  mongodb3:
    image: mongo
    container_name: mongodb3
    ports:
      - "27019:27017"
    networks:
      - mongo_network

  mongodb4:
    image: mongo
    container_name: mongodb4
    ports:
      - "27020:27017"
    networks:
      - mongo_network

  mongodb5:
    image: mongo
    container_name: mongodb5
    ports:
      - "27021:27017"
    networks:
      - mongo_network

  flask_app1:
    build: ./flask_app
    container_name: flask_app1
    networks:
      - mongo_network
    depends_on:
      - mongodb1
      - mongodb2
      - mongodb3
      - mongodb4
      - mongodb5

  flask_app2:
    build: ./flask_app
    container_name: flask_app2
    networks:
      - mongo_network
    depends_on:
      - mongodb1
      - mongodb2
      - mongodb3
      - mongodb4
      - mongodb5

  flask_app3:
    build: ./flask_app
    container_name: flask_app3
    networks:
      - mongo_network
    depends_on:
      - mongodb1
      - mongodb2
      - mongodb3
      - mongodb4
      - mongodb5

  flask_app4:
    build: ./flask_app
    container_name: flask_app4
    networks:
      - mongo_network
    depends_on:
      - mongodb1
      - mongodb2
      - mongodb3
      - mongodb4
      - mongodb5

  flask_app5:
    build: ./flask_app
    container_name: flask_app5
    networks:
      - mongo_network
    depends_on:
      - mongodb1
      - mongodb2
      - mongodb3
      - mongodb4
      - mongodb5

  nginx:
    image: nginx
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    networks:
      - mongo_network

networks:
  mongo_network:
    driver: bridge
```

# LOCUSTFILE.PY
```
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def slow_endpoint(self):
        self.client.get("/slow")

    @task
    def fast_endpoint(self):
        self.client.get("/fast")
```
