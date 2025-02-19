## LHCI SERVER AND DATABASE INSTALLATION

- Removing any old docker network:
`docker network rm lhci-network`

- Create a new docker network:
`docker network create lhci-network`

- Start the docker container for the LHCI MYSQL DB:
`docker rm lhci-mysql-db
docker run -d --name lhci-mysql-db --network lhci-network -p 3306:3306 -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_USER=tseronisk -e MYSQL_PASSWORD=kostas77 -e MYSQL_DATABASE=lhci -v /home/tseronisk/CODE/lhci-server/lhci-mysql-db-data:/var/lib/mysql mysql --bind-address=0.0.0.0`

- Login to the MYSQL DB docker instance to do the initial setup (password: pass):
`docker exec -it lhci-mysql-db mysql -uroot -ppass`

- Initial setup of the LHCI MYSQL DB:
`CREATE DATABASE lhci;
CREATE USER 'tseronisk'@'%' IDENTIFIED WITH mysql_native_password BY 'kostas77';
GRANT ALL PRIVILEGES ON lhci.* TO 'tseronisk'@'%';
FLUSH PRIVILEGES;`
`exit`

`docker restart lhci-mysql-db`

- Start the docker container for the LHCI SERVER:
`docker rm lhci-server
docker run -d -p 9001:9001 --name lhci-server --network lhci-network -v /home/tseronisk/CODE/lhci-server/lighthouserc-server.json:/usr/src/lhci/lighthouserc.json patrickhulce/lhci-server`

Here's what each component of the previous command does:
```
-it: This option ensures that Docker runs in interactive mode with a terminal, allowing you to interact with the LHCI wizard.
--rm: This option ensures that the Docker container is removed after you exit the interactive session. This is generally useful to keep your environment clean, especially when running containers in interactive mode for one-off commands.
-p 9001:9001: This binds port 9001 of your host machine to port 9001 of the container, which is the port LHCI server runs on.
-v /home/ec2-user/lighthouserc-server.json:/usr/src/lhci/lighthouserc.json: This mounts the lighthouserc.json file from your host machine to the specified path in the container. This ensures that when LHCI runs inside the container, it uses your configuration.patrickhulce/lhci-server: This is the name of the Docker image you're using.
```
- Remember to setup port forwarding on the router at this point, so the DB is accessible publically (e.g Grafana Cloud)


---------------------------------- FOLOWING STEPS NOT RELEVANT ANYMORE -------------------------------
Run the MySQL exporter for Prometheus

create a .my.cnf file for MySQL exporter
`nano ~/lhci-mysql-exporter.cnf`
```[client]
user=tseronisk
password=kostas77
host=lhci-mysql-db
port=3306
```

`docker run -d --name mysql-exporter --network lhci-network -p 9104:9104 -v ~/lhci-mysql-exporter.cnf:/etc/mysql/my.cnf prom/mysqld-exporter --config.my-cnf=/etc/mysql/my.cnf`

Verify it's running:
```docker ps
docker logs mysql-exporter
curl http://localhost:9104/metrics
```

Create a rometheus.yml file
```global:
  scrape_interval: 15s  # How often Prometheus collects metrics

scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']  # This must match the MySQL Exporter container name
```

Run Prometheus wih Docker:
```docker stop prometheus
docker rm prometheus
docker run -d --name prometheus --network lhci-network \
  -p 9090:9090 \
  -v ~/CODE/lhci-server/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```

---------------------------------- ABOVE STEPS NOT RELEVANT ANYMORE -------------------------------



Here's a rough outline of how you can manually create a new project using the Lighthouse CI server API:

    API Endpoint:
        The endpoint for creating a new project is usually POST /v1/projects.

    Sample cURL command:

    curl -X POST "http://localhost:9001/v1/projects" \
         -H "Content-Type: application/json" \
         -d '{
             "name": "My New Project",
             "externalUrl": "http://example.com",
             "slug": "my-new-project"
           }'

        Replace localhost with your server's address if it's not local.
        You can adjust the name, externalUrl, and slug as necessary.

    Response:
        The server should respond with a JSON payload that includes details about the project, including the token. This token is what you would use as the build token when running Lighthouse CI against this project.

---------------------------------------------------------------------------
### EXAMPLE API call for LHCI project creation

curl -X POST "http://localhost:9001/v1/projects" \
     -H "Content-Type: application/json" \
     -d '{
         "name": "eCommerce-B2C-pdp-mobile",
         "externalUrl": "",
         "slug": "eCom-B2C"
       }'

- **Responses for current LHCI projects:**

```
{"name":"eCommerce-B2C-pdp-desktop","externalUrl":"","slug":"ecommerce-b2c-pdp-desktop","baseBranch":"master","adminToken":"qzdluIprNJ1U3KJ2qddpSAk3fzozWlXhOMoqJq9S","token":"c5af7b19-3a0b-4b2e-a5e6-24c6133924fe","id":"02e42ca6-e6dd-46db-94f9-390dbf63e69a","updatedAt":"2025-02-19T13:28:45.787Z","createdAt":"2025-02-19T13:28:45.787Z"}
{"name":"eCommerce-B2C-home-desktop","externalUrl":"","slug":"ecommerce-b2c-home-desktop","baseBranch":"master","adminToken":"Ol2ZhM4jmalAhnkmdXyyKIzX7dFslZ3Nl94pWvQj","token":"6813feb2-6c86-4e6b-9040-e40f6b59c2c4","id":"3e63a681-1775-4f0c-b05e-324415b16332","updatedAt":"2025-02-19T13:29:21.082Z","createdAt":"2025-02-19T13:29:21.082Z"}
{"name":"eCommerce-B2C-home-mobile","externalUrl":"","slug":"ecommerce-b2c-home-mobile","baseBranch":"master","adminToken":"xqllCJBup6lSlM0OYgUA4mosMqHoP0RPN3Hljro2","token":"5d18c773-4aa1-418b-a6fd-73bc21458f67","id":"93137a60-e425-41af-9649-e2e14bf845d5","updatedAt":"2025-02-19T13:29:48.620Z","createdAt":"2025-02-19T13:29:48.620Z"}
{"name":"eCommerce-B2C-pdp-mobile","externalUrl":"","slug":"ecommerce-b2c-pdp-mobile","baseBranch":"master","adminToken":"fOw2WgIowFq5h6f0T2pzZ0zCxXKr8MtREu1ckWIW","token":"b8c469dc-3fca-4fca-a9cb-d0c15803741c","id":"37d289fb-da9e-4fa1-a4b9-26cf6a00b629","updatedAt":"2025-02-19T13:30:19.501Z","createdAt":"2025-02-19T13:30:19.501Z"}
```


## JENKINS SHELL SCRIPT

```
#!/bin/bash

# Generate a random hash/branch using Node.js
export LHCI_BUILD_CONTEXT__CURRENT_HASH=$(node -e "console.log(require('crypto').createHash('md5').update(Date.now().toString() + Math.random().toString()).digest('hex'))")
#export LHCI_BUILD_CONTEXT__CURRENT_BRANCH=$(node -e "console.log(require('crypto').createHash('md5').update(Date.now().toString() + Math.random().toString()).digest('hex'))")
export LHCI_BUILD_CONTEXT__CURRENT_BRANCH='main'

# Set commit message using current date/time
export LHCI_BUILD_CONTEXT__COMMIT_MESSAGE=$(date +"%Y-%m-%d %H:%M:%S")

# Set other lhci environment variables
export LHCI_BUILD_CONTEXT__AUTHOR='tseronisk'
export LHCI_BUILD_CONTEXT__AVATAR_URL='N\\A'
export LHCI_BUILD_CONTEXT__COMMIT_TIME=$(date -u +"%Y-%m-%dT%H:%M:%S.000Z")
#export LHCI_GITHUB_TOKEN='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'

# Ensure npm uses the correct path
export PATH=$HOME/.npm-global/bin:$PATH

sudo apt update
#sudo apt-get install fonts-liberation
#sudo apt install -y wget gnupg
#wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb 2>/dev/null
#sudo dpkg -i google-chrome-stable_current_amd64.deb

wget https://chromedriver.storage.googleapis.com/LATEST_RELEASE
export CHROMEDRIVER_VERSION=$(cat LATEST_RELEASE)
wget https://chromedriver.storage.googleapis.com/$CHROMEDRIVER_VERSION/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
sudo mv chromedriver /usr/local/bin/

google-chrome --version

ls -la
node -v
#npm install
npm install -g @lhci/cli@0.13.x
env
#sudo chmod 1777 /tmp

#npx lhci autorun
#lhci autorun --config=lighthouserc.json
npm run lhci-autorun-override-debug
```


jenkins workspace on locally running jenkins can be found in 'kostastseronis@mini-pc:~$ cd /var/lib/jenkins/workspace'
