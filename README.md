# My Experience Setting Up the MERN Stack on AWS EC2 with a Self-Hosted MongoDB

A **MERN stack** application is a full-stack web application built using four key technologies: **MongoDB**, **Express.js**, **React**, and **Node.js**. Each of these technologies serves a specific role in the development of the application:

1. **MongoDB**: A NoSQL database where the application stores its data in a flexible, document-based format (JSON-like documents).
   
2. **Express.js**: A web application framework for Node.js, used to build the server-side (backend) of the application. It handles routing, middleware, and HTTP requests.
   
3. **React**: A JavaScript library for building user interfaces, primarily on the client-side (frontend). React enables the creation of reusable UI components.
   
4. **Node.js**: A runtime environment that allows developers to run JavaScript on the server-side. It handles the server-side logic and communication with the database.

### How it works together:
- **MongoDB**: stores the application's data.
- **Express.js**: handles server-side routing and API requests.
- **React**: renders the user interface on the frontend.
- **Node.js**: manages the server-side processes, connecting the frontend and backend.

A typical MERN stack application would involve users interacting with the React frontend, which sends HTTP requests to the backend (Node.js and Express.js). The backend communicates with the MongoDB database to fetch or store data, then sends responses back to the frontend to update the user interface.

The MERN stack is popular for building web applications because it's based entirely on JavaScript, making development more seamless across the entire project, from frontend to backend.



# Prerequisites

Before embarking on this process, I made sure to have:

- An AWS EC2 instance running Ubuntu
- A basic understanding of Linux commands
- A MERN stack application prepared for deployment
i also ensured that ports 5000 and 27017 are open in the security group of my instance

# Installing MongoDB

I imported the public key for the MongoDB package:

`
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
`

Created a MongoDB source list file:

` 
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
`
Updated the package list and installed MongoDB:

`
sudo apt-get update
sudo apt-get install -y mongodb-org
`
Started MongoDB and enabled it to start on boot:

`
sudo systemctl start mongod
sudo systemctl enable mongod
`
Verified the installation:

` sudo systemctl status mongod `

# Configuring MongoDB 

## Setting up MongoDB for production was essential. Here’s what I discovered:
Configuring for remote access
` sudo nano /etc/mongod.conf`
Changed the bindIp setting:
`
net:
  bindIp: 0.0.0.0
  `
## Setting up a MongoDB user:

`mongosh`

![image](https://github.com/user-attachments/assets/0088a458-c84a-41c4-9917-607126777645)

`
use sample_todo_app
db.createUser({
  user: "sampleTodoUser",
  pwd: "Password.1",
  roles: [{ role: "readWrite", db: "sample_todo_app" }]
})
`
![image](https://github.com/user-attachments/assets/b3e241c4-0d88-43a6-be3c-5801a4bc01b8)

## Enabling authentication:

` sudo nano /etc/mongod.conf `

I Added an extral security 

`
 security:
  authorization: enabled
`

## The resulting connection string:

`
mongodb://sampleTodoUser:Password.1@localhost:27017/sample_todo_app?authSource=sample_todo_app
`


![image](https://github.com/user-attachments/assets/2a88e8ac-8141-4946-bc64-52129c445fa6)
.
# Installing Node.js

i made use of the following command when  Installing Node.js and it was straightforward:

`curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -`
![image](https://github.com/user-attachments/assets/719b1118-203a-43d9-a5b9-a85ce1826105)

.
`sudo apt-get install -y nodejs`
.
![image](https://github.com/user-attachments/assets/374ad9d3-5e55-4b7d-8a5f-05d4d1767be6)

.
to verify my installation i use this command 
`node -v`
`npm -v`

![image](https://github.com/user-attachments/assets/53461a7f-6432-4d47-a54e-c6891748270d)
.

# Setting Up PM2 Process Manager 

This was my first time using PM2, but I quickly realized its importance for production:
i used this  command  to install pm2 
`sudo npm install pm2 -g`

![image](https://github.com/user-attachments/assets/74cfded0-e0a2-47a0-9dc8-e20c4041a30b)

THe function of PM2’s is to help keep the application running after system reboots for maintaining uptime.

# Installing and Configuring Nginx

I installed Nginx with the following command 
`sudo apt update`
`sudo apt install nginx -y`

![image](https://github.com/user-attachments/assets/62d2ec2c-4d1f-406a-981d-a0f46b21d147)

I Created a custom Nginx configuration:

`sudo nano /etc/nginx/sites-available/sample_todo_app`

## Configuration 
`
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'Upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
`
![image](https://github.com/user-attachments/assets/a54fd84d-b091-46f1-ae4a-0346ef1be84b)

I activated the new configuration and deactivated the default using  the following commands 

`
sudo ln -s /etc/nginx/sites-available/sample_todo_app /etc/nginx/sites-enabled/
sudo nginx -t
sudo unlink /etc/nginx/sites-enabled/default
sudo systemctl restart nginx
`

![image](https://github.com/user-attachments/assets/f412d1f5-0fd4-4d29-94ef-5c2b4bb07ec7)

# Deploying the MERN Application

Finally, the moment of truth—deploying the application:

I followed this proceedure to deploy my MERN application 

Cloned the repository:

` git clone https://github.com/fmanimashaun/sample-todo-app.git `

Set up the frontend:
`cd sample-todo-app/client`
`sudo npm i && sudo npm run build`


