# Troubleshooting

## Backend

The best way to find out why the page is not being served is to SSH into your droplet as root and then to run ``docker ps``. You will see a list of running docker containers and  names for each. To find out what is wrong with the backend container, enter ``docker logs cupldeploy_cuplbackend_1``. This will print out log messages from the Flask application. If, for example, there is a problem connecting to the database, it will be reported here.
