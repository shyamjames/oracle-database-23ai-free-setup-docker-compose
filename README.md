# 🐘 Oracle Database 23c Free Docker Setup

This repository provides a straightforward `docker-compose.yml` to set up an Oracle Database 23c Free instance within a Docker container. It includes comprehensive instructions for getting started and troubleshooting common pitfalls across different operating systems.

## 🚀 Getting Started

Follow these steps to deploy and connect to your local Oracle Database.

### 📝 Setup Instructions

1.  **Obtain Files:**
    * You can clone this repository:
        ```bash
        git clone https://github.com/shyamjames/oracle-database-23ai-free-setup-docker-compose.git
        cd oracle-database-23ai-free-setup-docker-compose
        ```
    * Or, simply download the `docker-compose.yml` file into a new, empty directory on your system.

2.  **Ensure Docker & Docker Compose are Ready:**
    * **All Platforms:** Make sure Docker Desktop (for macOS/Windows) or Docker Engine (for Linux) and Docker Compose are installed and running.
        * **Arch based Linux distros:** If not already: `sudo pacman -S docker docker-compose && sudo systemctl enable docker --now && sudo usermod -aG docker $USER` (❗ **Log out and back in** after `usermod` for changes to take effect).
        * **Other Linux:** Refer to your distro's package manager.
        * **macOS / Windows:** Download and install [Docker Desktop](https://www.docker.com/products/docker-desktop/).

3.  **The `docker-compose.yml` File:**
    Ensure your `docker-compose.yml` in your project directory looks like this:

    ```yaml
    services:
      oracle-db:
        image: [container-registry.oracle.com/database/free:23.5.0.0](https://container-registry.oracle.com/database/free:23.5.0.0)
        container_name: oracle-db
        ports:
          - "1521:1521" # Standard Oracle Listener Port
          - "5500:5500" # Oracle EM Express/APEX HTTP Port
        environment:
          - ORACLE_PWD=YourStrongPassword # ⚠️ IMPORTANT: Change 'YourStrongPassword' to a strong, secure password!
          - ORACLE_CHARACTERSET=AL32UTF8
        volumes:
          - ./data:/opt/oracle/oradata # Persist database files on your host
        networks:
          - oracle-net
        restart: unless-stopped # Always restart the container if it stops

    networks:
      oracle-net:
        driver: bridge
    ```

4.  **Start the Database Container:**
    Navigate to the directory containing your `docker-compose.yml` in your terminal and execute:
    ```bash
    docker-compose up -d
    ```
    * The very first time you run this, Docker will download the large Oracle Free image. This can take significant time depending on your internet connection. ⏳
    * A `data` directory will be automatically created in your project folder for persistent storage.

5.  **Monitor Database Initialization:**
    It takes time for the database to fully initialize inside the container (5-15+ minutes, sometimes longer on first run). Keep an eye on the logs:
    ```bash
    docker logs -f oracle-db
    ```
    * Look for the crucial message: `DATABASE IS READY TO USE!` 🎉

### 🛠️ Common Issues & Troubleshooting

If your container exits or fails to initialize, these are the most common culprits:

1.  **`Cannot create directory "/opt/oracle/oradata/FREE".` or `[FATAL] Prepare Operation has failed.` (Volume Permissions):**
    This indicates the Oracle user inside the container lacks write permissions to your host's `./data` directory.

    * **Fix Steps:**
        1.  Stop and remove any failed container instances, and clean its associated volume data:
            ```bash
            docker-compose down -v
            ```
        2.  Manually delete the `data` directory from your host filesystem to ensure a fresh start:
            ```bash
            rm -rf data
            ```
        3.  **Apply Permissions Fix (Choose your OS):**
            * **Arch Linux:** You often need to explicitly grant write permissions to the created `data` directory:
                ```bash
                sudo chmod -R 777 data
                ```
            * **macOS / Windows:** This step is usually **not required** due to Docker Desktop's handling of file sharing. However, ensure Docker Desktop has necessary file sharing permissions for your project directory (Docker Desktop -> Settings -> Resources -> File Sharing).
        4.  Restart the container:
            ```bash
            docker-compose up -d
            ```
        5.  Re-monitor logs (`docker logs -f oracle-db`) until `DATABASE IS READY TO USE!`.

2.  **Container Exits with "No such file or directory" (Incorrect `ORACLE_HOME`/`ORACLE_BASE`):**
    This happens if your `docker-compose.yml` includes `ORACLE_BASE` or `ORACLE_HOME` environment variables. These paths are internal to older Oracle installations and not applicable to the streamlined Oracle Free Docker image.
    * **Fix:** Ensure these lines are removed or commented out in your `docker-compose.yml`. Then `docker-compose down -v` and `docker-compose up -d`.

### 🔌 Connecting to Your Database

Once your `oracle-db` container is `Up (healthy)` and the logs confirm `DATABASE IS READY TO USE!`, you can connect.

1.  **Install an Oracle Client (e.g., SQL\*Plus):**
    * **Arch Linux:** Install from AUR: `yay -S oracle-instantclient-sqlplus` (or your preferred AUR helper).
    * **macOS:** First, install [Homebrew](https://brew.sh/) (if you haven't already). Then, install the SQL\*Plus client: `brew install instantclient-sqlplus`.
    * **Windows:** Download the Oracle Instant Client Basic and SQL\*Plus packages from the [Oracle website](https://www.oracle.com/database/technologies/instant-client/downloads.html). Extract them and configure your system's PATH environment variable to include the Instant Client directory.

2.  **Connect via SQL\*Plus:**
    Open your terminal/command prompt and use the following command, replacing `YourStrongPassword` with the password you set in your `docker-compose.yml`:

    ```bash
    sqlplus SYSTEM/YourStrongPassword@localhost:1521/FREEPDB1
    ```
    * **Connection Details:**
        * **Host:** `localhost`
        * **Port:** `1521`
        * **Service Name (PDB):** `FREEPDB1` (This is the default PDB for Oracle Free).
        * **Username:** `SYSTEM` (or `PDBADMIN`; `SYS` if connecting `AS SYSDBA`).
        * **Password:** Your `ORACLE_PWD` from `docker-compose.yml`.

    * If successful, you'll see the `SQL>` prompt. 🎉

---

Inspired by [wilfriedago/oracle-database-23ai-free-setup-guide](https://github.com/wilfriedago/oracle-database-23ai-free-setup-guide)

---