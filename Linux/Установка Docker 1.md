## [Install Docker Desktop](https://docs.docker.com/desktop/setup/install/linux/ubuntu/#install-docker-desktop)

Recommended approach to install Docker Desktop on Ubuntu:

1. Set up Docker's package repository. See step one of [Install using the `apt` repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository).

2. Download the latest [DEB package](https://desktop.docker.com/linux/main/amd64/docker-desktop-amd64.deb?utm_source=docker&utm_medium=webreferral&utm_campaign=docs-driven-download-linux-amd64). For checksums, see the [Release notes](https://docs.docker.com/desktop/release-notes/).

3. Install the package using `apt`:
```console
 sudo apt-get update
```

```console
 sudo apt-get install ./docker-desktop-amd64.deb
```

Note

At the end of the installation process, `apt` displays an error due to installing a downloaded package. You can ignore this error message.

1. > ```text
    > N: Download is performed unsandboxed as root, as file '/home/user/Downloads/docker-desktop.deb' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)
    > ```
    
    By default, Docker Desktop is installed at `/opt/docker-desktop`.
    

The DEB package includes a post-install script that completes additional setup steps automatically.

The post-install script:

- Sets the capability on the Docker Desktop binary to map privileged ports and set resource limits.
- Adds a DNS name for Kubernetes to `/etc/hosts`.
- Creates a symlink from `/usr/local/bin/com.docker.cli` to `/usr/bin/docker`. This is because the classic Docker CLI is installed at `/usr/bin/docker`. The Docker Desktop installer also installs a Docker CLI binary that includes cloud-integration capabilities and is essentially a wrapper for the Compose CLI, at `/usr/local/bin/com.docker.cli`. The symlink ensures that the wrapper can access the classic Docker CLI.

## [Launch Docker Desktop](https://docs.docker.com/desktop/setup/install/linux/ubuntu/#launch-docker-desktop)

To start Docker Desktop for Linux:

1. Navigate to the Docker Desktop application in your Gnome/KDE Desktop.
    
2. Select **Docker Desktop** to start Docker.
    
    The Docker Subscription Service Agreement displays.
    
3. Select **Accept** to continue. Docker Desktop starts after you accept the terms.
    
    Note that Docker Desktop won't run if you do not agree to the terms. You can choose to accept the terms at a later date by opening Docker Desktop.
    
    For more information, see [Docker Desktop Subscription Service Agreement](https://www.docker.com/legal/docker-subscription-service-agreement). It is recommended that you also read the [FAQs](https://www.docker.com/pricing/faq).
    

Alternatively, open a terminal and run:

```console
 systemctl --user start docker-desktop
```

When Docker Desktop starts, it creates a dedicated [context](https://docs.docker.com/engine/context/working-with-contexts) that the Docker CLI can use as a target and sets it as the current context in use. This is to avoid a clash with a local Docker Engine that may be running on the Linux host and using the default context. On shutdown, Docker Desktop resets the current context to the previous one.

The Docker Desktop installer updates Docker Compose and the Docker CLI binaries on the host. It installs Docker Compose V2 and gives users the choice to link it as docker-compose from the Settings panel. Docker Desktop installs the new Docker CLI binary that includes cloud-integration capabilities in `/usr/local/bin/com.docker.cli` and creates a symlink to the classic Docker CLI at `/usr/local/bin`.

After you’ve successfully installed Docker Desktop, you can check the versions of these binaries by running the following commands:

```console
 docker compose version
Docker Compose version v2.39.4

```

```console
 docker --version
Docker version 28.4.0, build d8eb465

```

```console
 docker version
Client:
 Version:           28.4.0
 API version:       1.51
 Go version:        go1.24.7
<...>
```

To enable Docker Desktop to start on sign in, from the Docker menu, select **Settings** > **General** > **Start Docker Desktop when you sign in to your computer**.

Alternatively, open a terminal and run:

```console
 systemctl --user enable docker-desktop
```

To stop Docker Desktop, select the Docker menu icon to open the Docker menu and select **Quit Docker Desktop**.

Alternatively, open a terminal and run:

```console
 systemctl --user stop docker-desktop
```

## [Upgrade Docker Desktop](https://docs.docker.com/desktop/setup/install/linux/ubuntu/#upgrade-docker-desktop)

When a new version for Docker Desktop is released, the Docker UI shows a notification. You need to download the new package each time you want to upgrade Docker Desktop and run:

```console
 sudo apt-get install ./docker-desktop-amd64.deb
```