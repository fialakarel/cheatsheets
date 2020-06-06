# Docker

## Remove old Docker images
```bash
docker rmi -f $(docker images -q)
```

## Remove all exited containers
```bash
docker rm $(docker ps -q -f status=exited)
```

## Remove all Docker named volumes
```bash
docker volume rm $(docker volume ls -q -f dangling=true)
```

## Clone the existing volume
```bash
docker run \
  --rm -it \
  --volume conda_envs:/from \
  --volume $(date +"%Y%m%d%H%M%S_conda_envs"):/to \
  alpine \
  /bin/sh -c "cd /from ; cp -av . /to"
```
