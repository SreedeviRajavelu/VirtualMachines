## Difference between docker-compose up (with hyphen) and docker compose up -d (no hyphen between docker and compose )
 
### docker-compose up (with hyphen)
- uses old Docker Compose (v1) which is a python based tool
- starts containers in the foreground, so you will see all logs from all containers in your terminal
- if you stop the terminal (Ctrl C) , the containers are also stopped unless you add the -d (detached)
- this command is still supported, but **considered legacy** - Docker now recommends the plugin-based syntax  

### docker compose up -d (no hyphen between docker & compose)
- uses new Docker Compose (v2), which is now part of the docker CLI as plugin
- -d means detached mode:
- containers start in the background. 
- you get your terminal back immediately
- logs are not streamed to terminal but you can view it with `docker compose logs -f`