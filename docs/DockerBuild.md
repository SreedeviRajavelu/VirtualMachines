## Building Docker images

`docker build --network host --tag database .`

What each part does:

- `docker build`: command to build a Docker image

- `--network host`: uses host's network stack during the build process (allows the build to access network resources as if it were running directly on the host machine)

- `--tag database`: tags (names) the resulting image as "database"

- `.`: uses current directory as the build context (where Docker looks for the Dockerfile and any files it needs to copy)

  - `--network host`: this flag is often used when the build process needs to download packages or access network resources that might be blocked by Docker's default networking.
