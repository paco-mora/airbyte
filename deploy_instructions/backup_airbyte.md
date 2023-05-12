https://discuss.airbyte.io/t/how-to-import-export-airbyte-to-a-new-instance-docker-to-docker-deploy/3514

Some considerations:

This tutorials use the default airbyte-db credentials, pay attention is you have changed them;
This tutorial works for a docker to docker migration, I didn’t test or validate on kubernetes or docker to external-db (I’ll try in the future);
Try to use the same Airbyte platform version in both instances.
Generate the backup data
run docker ps and you must see Airbyte running. You must be able to access your instance locally or using a ssh tunnel.

Let’s stop all services and only start the database to prevent any new data/update in it.

docker-compose down to stop all services
docker-compose up -d db to only start the database
you can check with docker ps and should see only the database container running.

Let’s create the backup database file.
docker exec airbyte-db pg_dump -U docker airbyte > airbyte_backup.sql

It will generate a local SQL file. You can check the file size and compress if it is too big.

docker-compose down to stop the database again.

Recreate our instance
Copy/transfer the SQL file to your new server.

Go to your new server and start Airbyte using docker-compose up -d after the service is running let’s stop to rebuild the database with docker-compose down.

Start only the database docker-compose up -d db
Run the following commands:

docker exec airbyte-db psql -U docker -c 'drop database airbyte'; to drop the new database
docker exec airbyte-db psql -U docker -c 'create database airbyte with owner docker;'
cat airbyte.sql| docker exec -i airbyte-db psql -U docker -d airbyte to regenerate your database.
docker-compose up -d to restart Airbyte with your backup.
(need improvement) You must trigger all connections again to create the temporal workflow for them again. You can use the Airbyte API to do it.
Some useful links:

Backup/Restore a dockerized PostgreSQL database - Stack Overflow 3