---
sidebar_position: 7
---

# Docker

Instead of installing the toolchain manually, we are provided with a docker configuration, and this website will utilize it for ease of use.




# Initializing the docker instance

1. Ensure that your machine has access to `docker` and `docker-compose`.
2. Ensure you are in the root directory, with the *docker* folder.
3. Run the following command to set up the container in detached mode.
   ```bash
        docker-compose up -d
   ```
   <!-- *This will set up the container in detached mode.* -->

**You now have the docker container running in the background!**

## Compiling for the first time

Let's compile this firmware and get an idea of what that process looks like!

1. Run the following command:
   ```bash
      docker-compose exec dev make
   ```

   :::info
   If you run into an error referencing `mtar_end_data`, simply run the command again.
   This is a known bug.
   :::


   Your output should look something like this in the end:
   
   ```
      DFU      .obj/f7-firmware/firmware.dfu
      JSON     .obj/f7-firmware/firmware.json
make[1]: Leaving directory '/project/firmware'
2022-05-22 03:51:34,148 [INFO] Firmware binaries can be found at:
      dist/f7
   ```
   
   
### Your .dfu to flash is now located in `dist/f7/flipper-z-{target}-full-{suffix}.dfu`
