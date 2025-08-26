## What is a Docker .tgz file ?

In the Docker context, a `.tgz ` file is a **compressed archive of a Docker image** created with:

```
  docker save -o image.tar <image:tag>

  # then gzip it

  tar czf image.tgz image.tar
```

Or directly:

`docker save <image:tag> | gzip > image.tgz`

Inside the `.tgz`:
- it is basically a tarball of JSON metadata and filesystem layer tarballs

## To make modifications to Docker .tgz file
- load the `.tgz` file into Docker (`docker load`)
- run a container from it
- make changes
- `docker commit`
- `docker save`

## Do changes to .tgz persist? 
- if you only change inside the running container from image, these changes don't go back into the .tgz. They live in the container's writable layer.
- **To persist changes into a new `.tgz`:**
  1. Run a container from the image
  2. Make modifications
  3. Commit the container -> new image
  ```
  docker commit <container-id> myimage:modified
  ```
  4. Save it back:
  `docker save myimage:modified | gzip > myimage-modified.tgz`

 ## How .tgz files work in practice
 - `.tgz` = portable snapshot of a Docker image
 - you can transfer it, then use docker load < my image.tgz to load it into Docker on another host
 -  mostly for air-gapped environments (no Docker Hub access) or backups.

## To load the .tgz into Docker
  `docker load < myimage.tgz`

- makes the image available locally so you can run containers from it

## To create a .tgz from an existing image
`docker save myimage:tag | gzip > myimage.tgz`
- now you can copy over myimage.tgz to another machine and docker load it there




