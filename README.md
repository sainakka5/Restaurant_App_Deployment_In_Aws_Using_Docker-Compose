<h1>  RESTAURANT MENU APPLICATION SETUP AND DEPLOYMENT </h1>

Setting Up an AWS Instance Using Terraform:

As the first step in deploying the Restaurant Menu application, I created an AWS EC2 instance using Terraform. Terraform is an infrastructure-as-code tool that simplifies provisioning and managing cloud resources. Below is how I set up the AWS EC2 instance.

Applying Terraform:

1.	Initialize Terraform:
terraform init
This sets up Terraform and downloads the required AWS provider.
2.	Plan Infrastructure:
terraform plan
This command previews the infrastructure changes that Terraform will apply.
3.	Apply Changes:
terraform apply
After confirming the changes, Terraform creates the EC2 instance and outputs the public IP.

Connecting to the Instance:

Once the instance was created, I connected to it using SSH:

ssh -i "my_key_pair.pem" ubuntu@<instance_public_ip>

Using Terraform to create the AWS instance made the deployment process much faster and more manageable. By defining the instance configuration as code, I could easily recreate or modify the infrastructure whenever needed.

![image](https://github.com/user-attachments/assets/f5d1caa7-7a60-4808-be71-0a3097744c0c)


VERIFYING FILE TRANSFER:

Once the transfer was complete, I logged into the EC2 instance using SSH and navigated to the destination directory:

    ssh -i "my_key_pair.pem" ubuntu@<instance_public_ip>
    cd /home/root/Restaurant-App

I used the ls command to confirm that all files and folders were successfully copied. The directory contained:

    Dockerfile
    api
    config
    db.js
    docker-compose.yml
    middleware
    models
    node_modules
    package.json
    public
    server.js

![image](https://github.com/user-attachments/assets/a2eb1245-3e8c-4b10-92cd-ffb650f7e567)
 
Here I created a docker file by using my application with this I want to make a docker-compose file

![image](https://github.com/user-attachments/assets/3487a97d-4a09-42d0-9c3b-5d17048597f9)
 
Correct Database Host in Node.js Configuration:

In your current error, your Node.js application is trying to connect to 172.20.0.2:3306, which is an internal Docker network IP address. For containers to communicate with each other using service names defined in the docker-compose.yml file, you need to use the service name (mysql_db in your case) as the host, instead of an internal IP address.
Update your db.js or wherever the database connection is configured to use the service name (mysql_db) instead of the IP address.

Here’s how your db.js file should look:

    const mysql = require('mysql2');
    const dotenv = require('dotenv');
    
    dotenv.config();
    
    const pool = mysql.createPool({
        host: process.env.DB_HOST,
        user: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
    });
    
    pool.getConnection((err) => {
        if (err) throw err;
        console.log("Connected to MySQL Database");
    });
    
    module.exports = pool.promise();

Make sure to update your environment variables (.env file or the environment section in docker-compose.yml) to match the correct MySQL connection settings.

3. Ensure Containers are on the Same Network:
   
By default, docker-compose creates a default network for your services. However, you should verify that both your Node.js (app) and MySQL (mysql_db) containers are on the same network.

In my docker-compose.yml, ensure the containers are part of the same network (which they should be by default). Here’s how the docker-compose.yml file might look:

    version: '3.8'
    
    services:
      app:
        build: .
        container_name: restaurant-app
        ports:
          - "5000:5000"
        environment:
          DB_HOST: mysql_db  # Refer to MySQL service by its name here
          DB_USER: root
          DB_PASSWORD: root
          DB_NAME: restaurant_menu
        depends_on:
          - mysql_db  # Ensure app waits for mysql_db to be ready
        networks:
          - app-network
    
      mysql_db:
        image: mysql:8.0
        container_name: mysql_db
        environment:
          MYSQL_ROOT_PASSWORD: root
          MYSQL_DATABASE: restaurant_menu
        volumes:
          - mysql_data:/var/lib/mysql
        ports:
          - "3306:3306"
        networks:
          - app-network
    
    networks:
      app-network:
        driver: bridge
    
    volumes:
      mysql_data:
        driver: local

•	The key part is the DB_HOST: mysql_db in the app service.

•	The depends_on directive ensures that the Node.js application waits for the MySQL container to be ready before starting. Note that this only waits for the container to be up, not for MySQL to be fully ready. To ensure MySQL is ready, you might need additional health checks, but the basic depends_on should suffice for most cases.

4. Rebuild and Restart Containers
Once updated your Docker Compose and database connection settings, you can rebuild the containers and restart them:
docker-compose down  # Stop and remove all containers
docker-compose up --build  # Rebuild and start the containers

 ![image](https://github.com/user-attachments/assets/0157fd52-1598-407e-b350-210c633e12c2)

![image](https://github.com/user-attachments/assets/c56ca12d-beee-4bd0-81c7-1b3587c1fa9b)
 

After successfully run the docker container check the container with the below command.

 Sudo docker ps

 ![image](https://github.com/user-attachments/assets/777b5ab6-7f6c-43f5-9285-f32c99e3ee8f)

Accessing the Application:

After successfully running the Docker containers for my Node.js backend and MySQL database, I was able to access the Restaurant Menu application in my browser using the public IP address of my EC2 instance.
Accessing the Application in the Browser
With the containers running, I opened my web browser and accessed the application using the EC2 instance's public IP address and the specified port.

http://<instance_public_ip>:5000

If the application was accessible, I could see the home page or API responses from the backend.
Once the application was running and accessible in the browser, I confirmed that it was functioning as expected, including connecting to the database and serving API requests. This marked the successful deployment of my application on AWS using Docker containers!
 
 ![image](https://github.com/user-attachments/assets/468b33a2-d272-42ee-b4a9-954694fec8d9)

 ![image](https://github.com/user-attachments/assets/74dad317-3307-473a-9b8f-4d80c6cfb312)

![image](https://github.com/user-attachments/assets/095da294-ea0b-4c38-be1a-c32918c1220a)

![image](https://github.com/user-attachments/assets/858a7f52-a82d-44b0-a133-18155d9c59e8)

![image](https://github.com/user-attachments/assets/87cff07c-bc58-45dd-9814-0dc02a777b68)


After giving the information it will store them in the database server.
 
![image](https://github.com/user-attachments/assets/584c7e60-dcf1-4c06-9600-6a80b6f6a6c7)
