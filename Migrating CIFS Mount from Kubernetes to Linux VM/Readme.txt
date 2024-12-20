# Migrating CIFS Mount from Kubernetes to Linux VM

This guide explains how to migrate the setup of a CIFS (SMB) mount from Kubernetes to a Linux VM hosting Docker containers. The shared folder `//192.168.1.100/shared-folder` will be mounted directly on the Linux VM, and Docker containers will access it as a host-mounted volume.

---

## Prerequisites
1. **Linux VM** running Docker.
2. **CIFS Utilities** installed on the Linux VM.
3. **Access credentials** (username and password) for the CIFS share.
4. Ensure the Linux VM has network access to the CIFS share (`192.168.1.100`).

---

## Step 1: Install CIFS Utilities
Install the required CIFS utilities to enable SMB/CIFS support on the Linux VM:

```bash
sudo apt update
sudo apt install cifs-utils -y
```

---

## Step 2: Create a Mount Point
Create a directory on the Linux VM where the shared folder will be mounted:

```bash
sudo mkdir -p /mnt/shared-folder
```

---

## Step 3: Mount the CIFS Share
Mount the CIFS share using the `mount` command:

```bash
sudo mount -t cifs //192.168.1.100/shared-folder /mnt/shared-folder -o username=username,password=password
```

Replace:
- `username` with your actual CIFS share username.
- `password` with your CIFS share password.

### Verify the Mount
Check that the shared folder is accessible:

```bash
ls /mnt/shared-folder
```

You should see the contents of the shared folder.

---

## Step 4: Make the Mount Persistent (Optional)
To ensure the share is automatically mounted after a reboot, update the `/etc/fstab` file:

1. Open `/etc/fstab` in a text editor:

    ```bash
    sudo nano /etc/fstab
    ```

2. Add the following line at the end of the file:

    ```
    //192.168.1.100/shared-folder /mnt/shared-folder cifs username=username,password=password,uid=1000,gid=1000,iocharset=utf8 0 0
    ```

    Replace `username` and `password` with your credentials.

3. Save and close the file. Test the configuration by running:

    ```bash
    sudo mount -a
    ```

---

## Step 5: Share the Mount with Docker Containers
You can now share the mounted folder with your Docker containers.

### Option 1: Docker Run Command
Use the `-v` flag to bind mount the directory into the container:

```bash
docker run -d --name my-container -v /mnt/shared-folder:/app/shared-folder my-image
```

This command mounts `/mnt/shared-folder` from the host to `/app/shared-folder` inside the container.

### Option 2: Docker Compose
If using Docker Compose, define the volume in your `docker-compose.yml` file:

```yaml
version: '3.8'
services:
  my-service:
    image: my-image
    volumes:
      - /mnt/shared-folder:/app/shared-folder
```

Run the service:

```bash
docker-compose up -d
```

---

## Step 6: Test Access in the Container
Check if the shared folder is accessible from within the container:

```bash
docker exec -it my-container bash
ls /app/shared-folder
```

You should see the files from the CIFS share.

---

## Notes
- If the CIFS share requires specific SMB versions, include the `vers` option in the mount command or `/etc/fstab` entry:

    ```bash
    sudo mount -t cifs //192.168.1.100/shared-folder /mnt/shared-folder -o username=username,password=password,vers=3.0
    ```

- Ensure the Linux VM has sufficient permissions to access the CIFS share.

---

## Summary
This guide covers migrating a CIFS mount from Kubernetes to a Linux VM, including mounting the shared folder on the VM and sharing it with Docker containers. Follow these steps to ensure a seamless transition and continued access to the CIFS share from your application.

