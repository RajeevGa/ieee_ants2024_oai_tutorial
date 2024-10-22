# Procedure to pull and run OAI 5G core docker images 


```bash
cd ANTS_O-RAN_OAI_TUTORIAL/cn
```

If you dont have a docker account, create a docker account in [docker signup](https://www.docker.com/) 

Login to docker

```bash
docker login -u <username>
```

Enter your password

Now, pull the 5G core docker images

```bash
docker compose pull
```

strart core network
```bash
docker compose -f docker-compose.yml up -d
```

watch status of the core network
```bash
watch docker compose -f docker-compose.yml ps -a
```
All the docker containors should be `healthy`