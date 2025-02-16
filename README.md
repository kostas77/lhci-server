## LHCI SERVER AND DATABASE INSTALLATION

- Removing any old docker network:
`docker network rm lhci-network`

- Create a new docker network:
`docker network create lhci-network`


- Start the docker container for the LHCI MYSQL DB:
`docker run -d -p 3306:3306 --name lhci-mysql-db --network lhci-network -e MYSQL_ROOT_PASSWORD=pass -v /home/tseronisk/CODE/lhci-server/lhci-mysql-db-data:/var/lib/mysql mysql`

- Login to the MYSQL DB docker instance to do the initial setup (password: pass):
`docker exec -it lhci-mysql-db mysql -uroot -p`

- Initial setup of the LHCI MYSQL DB:
`CREATE DATABASE lhci;CREATE USER 'tseronisk'@'%' IDENTIFIED BY 'kostas77';GRANT ALL PRIVILEGES ON lhci.* TO 'tseronisk'@'%';FLUSH PRIVILEGES;`
`exit`

- Start the docker container for the LHCI SERVER:
`docker run -d -p 9001:9001 --name lhci-server --network lhci-network -v /home/tseronisk/CODE/lhci-server/lighthouserc-server.json:/usr/src/lhci/lighthouserc.json patrickhulce/lhci-server`

Here's what each component of the previous command does:
```
-it: This option ensures that Docker runs in interactive mode with a terminal, allowing you to interact with the LHCI wizard.
--rm: This option ensures that the Docker container is removed after you exit the interactive session. This is generally useful to keep your environment clean, especially when running containers in interactive mode for one-off commands.
-p 9001:9001: This binds port 9001 of your host machine to port 9001 of the container, which is the port LHCI server runs on.
-v /home/ec2-user/lighthouserc-server.json:/usr/src/lhci/lighthouserc.json: This mounts the lighthouserc.json file from your host machine to the specified path in the container. This ensures that when LHCI runs inside the container, it uses your configuration.patrickhulce/lhci-server: This is the name of the Docker image you're using.
```

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
         "name": "eCommerce-B2C-pdp-desktop",
         "externalUrl": "",
         "slug": "eCom-B2C"
       }'

- **Responses for current LHCI projects:**

```
{"name":"eCommerce-B2C-pdp-desktop","externalUrl":"","slug":"ecommerce-b2c-pdp-desktop","baseBranch":"master","adminToken":"l45ATleegclW5avKdhRAIUW4CClAuwJrMRCuCexX","token":"5a2b4932-6313-470c-9c79-982d06c0618d","id":"4918737b-68a3-4280-952a-b47089bc3517","updatedAt":"2025-02-16T15:34:30.913Z","createdAt":"2025-02-16T15:34:30.913Z"}
{"name":"eCommerce-B2C-home-desktop","externalUrl":"","slug":"ecommerce-b2c-home-desktop","baseBranch":"master","adminToken":"zSxQt4u4Y07Rn8ASbqRm9F9kQNEtaBloWLJms3FM","token":"c7f2d5b2-0fb7-4708-8435-ec3777096089","id":"1a324cda-8e2a-4632-a2f2-40ced62e7d3e","updatedAt":"2025-02-16T15:33:30.624Z","createdAt":"2025-02-16T15:33:30.624Z"}
{"name":"eCommerce-B2C-home-mobile","externalUrl":"","slug":"ecommerce-b2c-home-mobile","baseBranch":"master","adminToken":"ReCVIjwMydcaQqB58llKD3lrbg5wiBp3x1ZxhlTx","token":"e584afb8-62e0-4bbd-b6aa-77d4f1e17446","id":"ed7336f7-d190-4b93-aa7a-6977214fdb43","updatedAt":"2025-02-16T15:32:59.389Z","createdAt":"2025-02-16T15:32:59.389Z"}
{"name":"eCommerce-B2C-pdp-mobile","externalUrl":"","slug":"ecommerce-b2c-pdp-mobile","baseBranch":"master","adminToken":"8JrzADhWRx0B24WzgjH2uL5tbo0A3NzU26QAtkhs","token":"6d5fe580-5999-464e-9d87-1aafb6cefd62","id":"c19e4c91-bfea-48b1-a57f-b9af9563a9aa","updatedAt":"2025-02-16T15:31:42.755Z","createdAt":"2025-02-16T15:31:42.755Z"}
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
