# Self-Hosted Odoo ERP Instance with Celonis Process Mining Integration

This guide will show you how to create a self-hosted Odoo instance using Docker and establish a connection with Celonis EMS for Process Mining.

> **Note:** This guide was inspired by [kaztakata/celonis-postgres](https://github.com/kaztakata/celonis-postgres/tree/main), with some improvements to make it work smoothly. It’s still a work in progress, so feel free to contribute or open issues for any questions or improvements!

## Prerequisites

- Docker & Docker Compose installed
- Access to Celonis EMS
- Basic understanding of Odoo and PostgreSQL


## Steps

### 1. Download the JDBC On-Premise Extractor

- **Login to Celonis EMS**.
- Go to **Admin & Settings** → **Download Portal** → **JDBC (Database) Extractor**.
- Download the `yyyy-mm-dd-dockerized-package-jdbc-2.103.0.zip` (Current Version).
- Extract the folder from the ZIP file.
- Load the Docker image:

   ```bash
   docker load -i <filename>.tar
   ```
   Replace <filename> with the actual name of the .tar file in the extracted folder.

### 2. Setup the Uplink
- In Celonis EMS, navigate to **Admin & Settings** → **Uplink Integrations**.
- Click **Connect New System**.
- Choose any name and select **Connector**.
- Copy the ClientID and ClientSecret tokens.
- Complete the process. The status should show a red X (this will turn green later).

### 3. Local adjustments
- Replace the image name in your docker-compose.yml file with the extractor image name you just downloaded. Ensure you include the version:
```yaml
image: connector-jdbc-on-prem:2.103.0
```
- In uplink.env, update the tokens with the values you copied from the previous step:
```bash
CLIENT_ID=<your-client-id>
CLIENT_SECRET=<your-client-secret>
UPLINK_URL=https://<your-celonis-environment-url>
```
**If you're in a productive environment, don’t forget to change the default passwords!**

### 4. Start Containers and verify status
- Start the Containers
```bash
docker compose up
```
- Check the Celonis EMS dashboard – your Uplink connection status should turn green if everything is set up correctly.

### 5. Login to PGAdmin
- Use your credentials to login to PGAdmin
- Add a new server connection. You can find the hostname of your Postgres container using:
```bash
docker inspect <postgres-container-token> | grep IPAddress
```
The default credentials for the database are:
- Username: odoo 
- Password: password

### 6. Setup Odoo and Enter some Data 
- Visit **localhost:8089** to access Odoo
- Setup Odoo using any input that you want. (Odoo will create a new database)
- Select Demo data if you want preloaded data
- Install apps and customize the mock data as needed

### 7. Establish data connection
- Make sure port **5432** is accessible from outside to allow access to your PostgreSQL database.
- In the Celonis EMS, go to **Data** → **Data Integration**
- Create a new **Data Pool**
- Inside the Data Pool, go to **Data Connections**
- Add a new **Data Connection** → **Connect Data Source**
- Choose **On-Premise** and select your Uplink-Configuration
- Enter the required information:
```bash
Database Type: PostgreSQL
Host: Your public IP address
Credentials: As set earlier during the setup
```

For more details, see the [Celonis Documentation](https://docs.celonis.com/en/connecting-to-a-database.html)

### 8. Stopping the instance
- To stop the containers, run:
```bash
docker compose down
```
- If you wish to delete all data (and restart Odoo from scratch), remove the volumes:
```bash
docker compose down -v
```
This will require you to reconfigure Odoo from the beginning.

## Conclusion
By following this guide, you have successfully created a self-hosted Odoo instance, connected it to Celonis EMS for process mining, and set up data integration. Feel free to experiment with different apps, data, and processes within Odoo to optimize your process mining tasks in Celonis.

If you encounter any issues or have suggestions for improvement, please open an issue in this repository.

## Helpful links
- [Celonis Documentation](https://docs.celonis.com/en/connecting-to-a-database.html)
- [Odoo Documentation](https://www.odoo.com/documentation/17.0/administration/on_premise.html)