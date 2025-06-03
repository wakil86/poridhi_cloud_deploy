

#  Setup Instructions
## 1. Open VS code on Poridhi`s lab and launch a terminal 

## 2. Database setup

### Install PostgreSQL
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib libpq-dev build-essential
```
### Active PosrgreSQL Database
```bash
sudo systemctl start postgresql
```
### Access PostgreSQL as the ```postgres``` user:
```bash
sudo -i -u postgres
psql
```
You should now be at the ```postgres=#``` prompt.

### Create your database
```bash
CREATE DATABASE flask_jwt_db;
```
### Create database user
```bash
CREATE USER poridhi WITH PASSWORD 'poridhi1234';
```
### Grant privilage to user
```bash
GRANT ALL PRIVILEGES ON DATABASE flask_jwt_db TO poridhi;
```
### Grant schema-level privilage 
```bash
\c flask_jwt_db; 
GRANT CREATE ON SCHEMA public TO poridhi;
```
- This allows ```poridhi``` to create tables within the ```public``` schema.

### Exit ```psql``` and the ```postgres``` user session:
```bash
\q
exit
```



## 3. Clone the Repository

```bash
git clone https://github.com/wakil86/Flask_JWT_postgresql.git
cd Flask_JWT_postgresql
```


## 4. Update & Install python3 & pip
```bash
sudo apt update
sudo apt install -y python3 python3-pip
```


## 5. Install Python dependencies
```bash
pip install -r requirements.txt
```
## 6. Run the Flask app
```bash
python3 app.py
```
### You will see the app is running

![APP_RUN](images/run.png)

- Here 5000 is the port which you can change from the ```app.py``` script

### Open another terminal & run ```ifconfig```

![APP_RUN](images/ifconfig.png)

- Copy the IP from ```eth0``` part.

***Now we will expose the IP using a load balancer***

### Create a load balancer
![APP_RUN](images/loadb.png)

- Make sure you are using right port as you mentioned on your ```app.py``` script.

### After creating load balancer you will get a link

![APP_RUN](images/loadb1.png)

- click on the link

### You will be redirected to this page.


![APP_RUN](images/copy.png)

- Copy the URL.

### Go to Postman app & paste the URL

![APP_RUN](images/working.png)

## Now you can test your endpoints 

###  Registration

```
POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/register
{
  "username": "testuser",
  "password": "testpass1234"
}
```
### Expected output
![APP_RUN](images/register.png)

###  Login

```
POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/login
{
  "username": "testuser",
  "password": "testpass1234"
}
```
### Expected output
![APP_RUN](images/login.png)

###  Protected Route

```
GET https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/protected/dashboard
Headers:
  Authorization: Bearer <JWT_TOKEN>
```
### Expected output
![APP_RUN](images/protected.png)

###  Logout

```
POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/logout
Headers:
  Authorization: Bearer <JWT_TOKEN>
```
### Expected output
![APP_RUN](images/logout.png)
---

## Now we will try this with docker
 
### Create a ```Dockerfile``` & ```docker-compose.yml``` script

```Dockerfile```
```bash
FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python3", "app.py"]
```

```docker-compose.yml```
```bash
version: '3.8' 

services:
  
  api:
    build: . 
    ports:
      - "5000:5000" 
    volumes:
      - .:/app 
    environment:
      
      DATABASE_URL: postgresql://poridhi:poridhi1234@db:5432/flask_jwt_db
      
      FLASK_APP: app.py
      FLASK_ENV: development 
    depends_on:
      - db 
    command: python3 app.py 

  
  db:
    image: postgres:13 
    restart: always 
    environment:
      POSTGRES_USER: poridhi
      POSTGRES_PASSWORD: poridhi1234
      POSTGRES_DB: flask_jwt_db
    ports:
      - "5432:5432" 
    volumes:
      - postgres_data:/var/lib/postgresql/data


volumes:
  postgres_data:
```
### Install ```Docker``` & ```Docker-Compose```

### Stop the ```postgresql``` server which is running on port 5432
```bash
sudo systemctl stop postgresql
```

### Build your docker image
```bash
docker compose up --build -d
```
### Expected output
![APP_RUN](images/dockerbuild.png)

### Run the containers
```bash
docker compose up
```
### Expected output
![APP_RUN](images/docrun.png)

***Your app is running.You can test this via terminal***

### Registration
```bash
curl -X POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/register \
-H "Content-Type: application/json" \
-d '{"username": "testuser", "password": "testpass"}'
```
### Expected output
![APP_RUN](images/registercurl.png)

### Login

```bash
curl -X POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/login \
-H "Content-Type: application/json" \
-d '{"username": "testuser", "password": "testpass"}'
```
### Expected output
![APP_RUN](images/logincurl.png)

### Protected Login
```bash
curl -X GET https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/protected/dashboard \
-H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTc0ODk3NTAyMCwianRpIjoiNzEwNDU3MTMtZDMwNy00YTk2LWFhNjYtNWYxZTRmOWFiN2NjIiwidHlwZSI6ImFjY2VzcyIsInN1YiI6InRlc3R1c2VyIiwibmJmIjoxNzQ4OTc1MDIwLCJjc3JmIjoiYzlkZmJmN2MtYjRhZC00YzQ4LTliOWEtMGJmZmM1ZjMzZDMyIiwiZXhwIjoxNzQ4OTc4NjIwfQ.zvfyo-qXdejvlT5OcG_2wokHnoQwSMEJmVs-sEILh3w"

```
### Expected output
![APP_RUN](images/protectedcurl.png)

### Logout
```bash
curl -X POST https://67aa6cbb465a4877519ba06d-lb-371.bm-east.lab.poridhi.io/auth/logout \
-H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJmcmVzaCI6ZmFsc2UsImlhdCI6MTc0ODk3NTAyMCwianRpIjoiNzEwNDU3MTMtZDMwNy00YTk2LWFhNjYtNWYxZTRmOWFiN2NjIiwidHlwZSI6ImFjY2VzcyIsInN1YiI6InRlc3R1c2VyIiwibmJmIjoxNzQ4OTc1MDIwLCJjc3JmIjoiYzlkZmJmN2MtYjRhZC00YzQ4LTliOWEtMGJmZmM1ZjMzZDMyIiwiZXhwIjoxNzQ4OTc4NjIwfQ.zvfyo-qXdejvlT5OcG_2wokHnoQwSMEJmVs-sEILh3w"
```
### Expected output
![APP_RUN](images/logoutcurl.png)




