# Docker Day-to-Day

This guide covers routine Docker operations, cleanup, troubleshooting, and container lifecycle tasks.

## 11.1 Container lifecycle

### List containers

```bash
docker ps
docker ps -a
```

### Start or stop or restart

```bash
docker start web
docker stop web
docker restart web
```

### Logs

```bash
docker logs web --tail 100
docker logs -f web
```

### Exec into container

```bash
docker exec -it web /bin/sh
docker exec -it web /bin/bash
```

### Inspect container

```bash
docker inspect web
```

## 11.2 Image management

### Pull image

```bash
docker pull nginx:stable
```

### Build image

```bash
docker build -t myapp:1.0.0 .
```

### Tag and push

```bash
docker tag myapp:1.0.0 registry.example.com/myapp:1.0.0
docker push registry.example.com/myapp:1.0.0
```

### Remove unused images

```bash
docker image prune -af
```

### Show disk usage

```bash
docker system df
```

## 11.3 Docker Compose operations

```bash
docker compose up -d
```

```bash
docker compose ps
```

```bash
docker compose logs -f
```

```bash
docker compose pull
```

```bash
docker compose up -d --force-recreate
```

```bash
docker compose down
```

## 11.4 Container resource monitoring

```bash
docker stats --no-stream
```

```bash
docker top web
```

```bash
docker inspect -f '{{.State.Status}} {{.State.RestartCount}}' web
```

## 11.5 Debugging containers

### Check environment

```bash
docker exec web env | sort
```

### Check mounted volumes

```bash
docker inspect -f '{{json .Mounts}}' web | jq .
```

### Check network settings

```bash
docker inspect -f '{{json .NetworkSettings.Networks}}' web | jq .
```

### Check recent exits

```bash
docker ps -a --filter status=exited
```

### Capture logs from all containers

```bash
for c in $(docker ps --format '{{.Names}}'); do echo "==== $c ===="; docker logs --tail 50 "$c"; done
```

## 11.6 Cleanup tasks

```bash
docker container prune -f
docker network prune -f
docker volume prune -f
```

## 11.7 Common operational tasks

### Copy file into container

```bash
docker cp config.yaml web:/etc/myapp/config.yaml
```

### Copy file out of container

```bash
docker cp web:/var/log/app.log ./app.log
```

### Run one-off command

```bash
docker run --rm alpine:3.20 date
```

## 11.8 Docker troubleshooting checklist

- Check restart count.
- Check entrypoint and command.
- Check image tag.
- Check env vars.
- Check secrets mounts.
- Check port mappings.
- Check disk usage.
- Check daemon logs.
- Check host firewall.

### Docker daemon logs

```bash
journalctl -u docker --since '1 hour ago' --no-pager
```

## 11.9 Docker one-liners

```bash
docker ps --format 'table {{.Names}}	{{.Status}}	{{.Image}}	{{.Ports}}'
```

```bash
docker images --format 'table {{.Repository}}	{{.Tag}}	{{.Size}}	{{.CreatedSince}}'
```

---
