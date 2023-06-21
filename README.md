# Nicotine-plus + noVNC Docker Container

![Docker Pulls](https://shields.api-test.nl/docker/pulls/cproensa/nicotine-docker)
![Docker Image Size](https://shields.api-test.nl/docker/image-size/cproensa/nicotine-docker)

This build is based on [realies/soulseek-docker](https://github.com/realies/soulseek-docker), adapted to run Nicotine-plus.

His documentation has more details and most of those are also applicable here.

## Setup

1. **You will need to map port 6080 on the machine (or any other port) to port 6080 on the docker container running this image.**
    * If you are using a GUI or webapp (e.g. Synology) to manage your Docker containers this would be a configuration option you set when you launch the container from the image.  
    * With the Docker CLI the option is `-p 6080:6080`.
1. **Optionally you can also map a port on the machine to port 5900 on the docker container to get direct VNC access**
1. **You will need to map a port for Nicotine listening port on the Docker container.**  These ports can be configured from within Nicotine but whatever those ports are, they need to be mapped a) from your router to the machine hosting this docker image and b) from the outside of the docker image to the server within it.  See below for more details.
1. **You will probably also want to set up a place on the local disk for Nicotine to work with/download to/etc.**  While you can of course just point the app at existing folders it is probably wiser to give the app its own siloed off location on disk. 

Once that is done you should be able to connect to the machine on port 6080 with a standard web browser through the magic of `noVNC`.  Example: if your docker VM machine has IP `192.168.1.23` you should be able to connect to the Nicotine app running in docker by typing `https://192.168.1.23:6080` (or `http://192.168.1.23:6080` depending on your machine's security settings) in your browser after launching the container.


## Usage
### Configuration Parameters

```
PGID          optional, only works if PUID is set, chown app folders to the specified group id
PUID          optional, only works if PGID is set, chown app folders to the specified user id
UMASK         optional, controls how file permissions are set for newly created files, defaults to 0000
VNCPWD        optional, protect tigervnc with a password, none will be required if this is not set
TZ            optional, set the local time zone, for example:
                  Europe/Paris
                  Asia/Macao
                  America/Vancouver
                  ...other values available in /usr/share/zoneinfo
```

### How To Launch
##### Using Docker Compose

```
docker-compose up -d
```

##### Using Docker Compose YML File

```
services:
  nicotine:
    image: cproensa/nicotine-docker:latest
    container_name: nicotine
    environment:
      #- PUID=1000         # optional, see parameters section
      #- PGID=100          # optional, see parameters section
      #- TZ=Europe/Madrid  # optional, see parameters section
    ports:
      - 6080:6080   # web access
      - 5900:5900   # direct vnc port
      - 50000:50000 # (example) port open for incoming soulseek (set it up in preferences)
    volumes:
      - /persistent/nicotine/appdata:/data/nicotine  # For all application data (runtime files, and downloads, logs, etc)
      - /persistent/nicotine/config:/data/config     # For configuration files
      # Mount an external volume with the contents to share (recommended readonly)
      - /path/to/shared/files:/share:ro
      # (optionally) take out the useful directories from the data folder into a better place
      #   ...these are the default directories, maped 1 by 1:
      # - /your/home/nicotine/complete:/data/nicotine/downloads
      # - /your/home/nicotine/incomplete:/data/nicotine/incomplete
      # - /your/home/nicotine/uploads:/data/nicotine/received
      # - /your/home/nicotine/logs:/data/nicotine/logs
      # or just mount a single external volume and change those directories in preferences to be inside that mount point:
      # - /your/home/nicotine:/folders
    restart: unless-stopped    
```

### How To Build

The Dockerfile script will copy an external .deb package during the build process, named `debian-package.zip`, which is the release package as described in the 
![Nicotine-plus documentation](https://github.com/nicotine-plus/nicotine-plus/blob/master/doc/DOWNLOADS.md#ubuntudebian)

Straightforward build:

    ```
    docker-compose build
    ```

##### Using build.sh script

`Usage: ./build.sh <target> [push-user]`

*target* can be either:

  - *latest* to download the latest stable .deb release version
  - *testing* to donwload the latest nightly build .deb package
  - *a specific version number* to download any version release .deb package (e.g. 3.2.8)

[optional] *push user* is the user name to push image to the hub.

  - Example: build locally with development release: `./build testing`
     - This will create a tagged image: `nicotine-docker:testing` 
  - Example: build with a version and push to hub: `./build 3.2.8 user_name`
     - This will create a tagged image: `user_name/nicotine-docker:3.2.8` and push.

