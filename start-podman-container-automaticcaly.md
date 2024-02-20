H0w To **automatically start Podman containers** after your host reboots, you can follow these steps:

1. **Get the container up and running**:
   - Start your desired container using the `podman run` command. For example, if you have a container named `chitragupta-db`, you can start it like this:
     ```
     $ podman run --name chitragupta-db docker.io/library/mariadb:latest
     ```
   - Note down the container name (in this case, `chitragupta-db`).

2. **Create a systemd service**:
   - Since Podman is daemon-less, we'll create a systemd unit file to manage the container startup.
   - Generate the systemd unit file for your container using the following command (replace `CONTAINER_NAME` with your actual container name):
     ```
     $ podman generate systemd --new --name CONTAINER_NAME
     ```
   - This command will create a systemd service unit file for your container.

3. **Enable and start the service**:
   - Move the generated unit file to the appropriate location (usually `/etc/systemd/system/`):
     ```
     $ sudo mv CONTAINER_NAME.service /etc/systemd/system/
     ```
   - Reload systemd to pick up the changes:
     ```
     $ sudo systemctl daemon-reload
     ```
   - Enable the service to start on boot:
     ```
     $ sudo systemctl enable CONTAINER_NAME.service
     ```
   - Start the service immediately:
     ```
     $ sudo systemctl start CONTAINER_NAME.service
     ```

Now your Podman container will automatically start after your server reboots. 🚀

Source: Conversation with Bing, 20/2/2024
(1) How to Autostart Podman Containers? - Linux Handbook. https://linuxhandbook.com/autostart-podman-containers/.
(2) Configure a container to start automatically as a systemd service. https://www.redhat.com/sysadmin/container-systemd-persist-reboot.
(3) How to Start a Container on Boot with Podman and Systemd. https://www.tutorialworks.com/podman-systemd/.
