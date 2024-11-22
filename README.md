# Custom Linuxserver-Duplicati Image with Docker CLI
This Dockerfile extends the [LinuxServer.io Duplicati](https://docs.linuxserver.io/images/docker-duplicati/) image by adding the Docker CLI.

The addition of the Docker CLI enables the container to interact with the host Docker engine, facilitating efficient container management through scripts. This makes it possible to create before and after scripts for Duplicati, such as generating database dumps or stopping and starting containers to ensure they are in a consistent state.

## Notes
For communication with Docker on the host machine, it is important to mount the Docker socket: /var/run/docker.sock:/var/run/docker.sock into the duplicati container.

## Example Docker-Compose Setup
```
name: backup
services:
  duplicati:
    image: cgraumann/linuxserver-duplicati-with-docker-cli:v1.0.0
    container_name: duplicati
    environment:
      - PUID=0
      - PGID=0
      - TZ=Europe/Berlin
    volumes:
      # Configuration for Duplicati
      - duplicati-config:/config

      # Folder for before and after scripts
      - duplicati-scripts:/scripts

      # Enable interaction with Docker CLI for before/after scripts
      - /var/run/docker.sock:/var/run/docker.sock

      # Source directories to back up
      - /var/lib/docker/volumes:/source/docker-volumes/

      # Backup destination
      - D:\backups:/backups
    ports:
      - 8200:8200
    restart: unless-stopped

volumes:
  duplicati-config:
  duplicati-scripts:

```

## Before and After Scripts

To create before and after scripts for Duplicati, place them in the following directory:
```
/scripts/
```

### Step-by-Step Instructions:

1. **Copy Script from Host to Container**  
   Use the following command to copy the script from the host machine into the container:
   ```bash
   docker cp ./script.sh duplicati:/scripts/script.sh
   ```

2. **Make the Script Executable**  
   After copying the script, make it executable inside the container:
   ```bash
   docker exec duplicati chmod +x /scripts/script.sh
   ```

3. **Configure Duplicati GUI**  
   In the Duplicati GUI, select you backup and navigate to the **"Advanced Options"** section:
   - Go to **Configuration → edit → 5. Options → Options for Pros**.
   - Add run-script-before
   - Enter the path of your script: `/scripts/script.sh`
     - ...

### Ensure the Script Runs Only During Backups:
To ensure that the script is only triggered during a backup and not during a restore operation, add the following condition in the script:

```bash
if [[ "${DUPLICATI__OPERATIONNAME,,}" != "backup" ]]; then
    exit 0
fi
```

This ensures the script exits early if the operation is not a backup (e.g., during a restore).

## Related
- [DockerHub](https://hub.docker.com/repository/docker/cgraumann/linuxserver-duplicati-with-docker-cli/general)
