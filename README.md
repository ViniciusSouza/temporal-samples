# UNDER CONSTRUCTION!

We're in the process of porting the Cadence samples over to Temporal.  These samples don't work yet.

## temporal-samples

This repository holds samples of workflows and activities written in .NET for the Temporal platform.  These have been tested on Windows and Mac OS/X using Visual Studio.  We'll have you run these workflows against Temporal/Cassandra running locally as a Docker container.

### Requirements

You'll need a computer configured like:

* A 64-bit Windows computer running Windows 10 with Hyper-V or a Mac running a recent version of OS/X.
* 4+ CPU cores
* Visual Studio 2019
* 8GB+ RAM
* Docker Desktop

Note that you'll need to enable Hyper-V for Windows because Docker requires it.  The Docker installation link explains how to accomplish this.

### Configuring Windows

1. Download and install **Visual Studio 2019**.  The **Community Edition** works fine and is free for most users, but any edition will work:

   [https://visualstudio.microsoft.com/vs/](https://visualstudio.microsoft.com/vs/)

2. Install Docker Desktop (community) by following the instructions here:

   [https://docs.docker.com/docker-for-windows/install/](https://docs.docker.com/docker-for-windows/install/)

3. Make sure you start Docker after installing it.

4. Increase Docker CPUs to 4 and RAM allocation to 4GB.

   a. **Right-click the Docker icon** in your system tray and select **Settings**
   b. Click **Advanced** on the left and then drag the **CPU slider to 4** and the **Memory slider to 4096**
   c. Click **Apply**

5. Configure Docker networking: We've never had much with the default Docker DNS network settings.  We recommend configuring Docker to use Google Public DNS (8.8.8.8):

   a. **Right-click the Docker icon** in your system tray and select **Settings**
   b. Click **Network** on the left
   c. Select **Fixed** in the **DNS Server** section and enter **8.8.8.8** as the server address.
   d. Click **Apply**

### Remarks

You may be able to adjust the Docker RAM allocation lower.  We encountered stablity issues with the default 2GB allocation and doubled this to 4GB for our purposes with good results.  There needs to be enough RAM to run the Docker Linux distribution along with the Temporal and Cassandra services.

On Windows, Docker always consumes the configured RAM, even if it's not actually doing anything.  We develop on Windows workstations with 32-64GB of RAM so the loss of 4GB doesn't bother us, but this can impact lesser machines.  You may want to exit and disable Docker when your not using it.  Use the **Settings/General** to disable Docker startup.

The current Docker on Windows implementation is hack compared to how Docker works natively on Linux and OS/X.  **Windows Docker is somewhat unstable** and we find it can lose network connectivity every few days.  Image pulls and image builds will fail and executing containers will be unable to establish network connections.  You can mitigate this by **right-clicking** on the Docker icon in the system tray and selecting **Restart...*  Docker is working on new implementation that takes advantage of the Windows Subsystem for Linux (WSL) which will eventually address these disavantages.

### Configuring Macintosh

1. Download and install **Visual Studio 2019 for Mac**:

   [https://visualstudio.microsoft.com/vs/mac/](https://visualstudio.microsoft.com/vs/mac/)

2. Install **Docker for Mac**:

   https://docs.docker.com/docker-for-mac/install/

3. Increase Docker CPUs to 4 and RAM allocation to 4GB.

   a. **Right-click the Docker icon** in your system tray and select **Settings**
   b. Click **Advanced** on the left and then drag the **CPU slider to 4** and the **Memory slider to 4096**
   c. Click **Apply**

4. Create a local file called **temporal.yml** that includes the following Docker stack/compose definition:
```
version: '3.5'

services:
  cassandra:
    image: cassandra:3.11
    ports:
      - "9042:9042"
  temporal:
    image: temporalio/auto-setup:0.21.1
    ports:
      - "7233:7233"
    environment:
      - "CASSANDRA_SEEDS=cassandra"
      - "DYNAMIC_CONFIG_FILE_PATH=config/dynamicconfig/development.yaml"
    depends_on:
      - cassandra
  temporal-web:
    image: temporalio/web:0.21.1
    environment:
      - "TEMPORAL_GRPC_ENDPOINT=temporal:7233"
    ports:
      - "8088:8088"
    depends_on:
      - temporal
```

5. Then you can start or stop the Temporal stack via:
   ```
# Start the Temporal stack:

docker stack deploy -c temporal.yml temporal-dev

# Stop/remove the Temporal stack:

docker stack rm temporal-dev
   ```

Note that it takes Temporal and Cassandra a several seconds to initialize.  Once Temporal has started, you can view its web UI via: [http://localhost:8088](http://localhost:8088)

## TemporalFixture: Easy Temporal unit testing

For those of you using [Xunit](https://github.com/xunit/xunit) for unit testing, we make the [TemporalFixture](https://doc.neonkube.com/T_Neon_Xunit_Temporal_TemporalFixture.htm) test fixture available.  Add this to your unit tests to automatically spin up a local Temporal server instance while your tests are running.

This repository includes examples demonstrating how to do this.
