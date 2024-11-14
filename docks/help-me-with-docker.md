# Instruction

## docker installation


## Skopeo hot commands
- login to docker repo
```bash
skopeo login docker.io/kostakoff
```
- copy docker image
```bash
skopeo copy docker://docker.io/kostakoff/rocky-base-images:8-default docker://docker.io/kostakoff/test:8-default
```
- create container with bash session and mount current folder for work
```bash
docker run -v $(pwd):/home/user -w /home/user --user $(id -u):$(id -g) --rm -it --entrypoint bash docker.io/kostakoff/rocky-base-images:8-python3.9
```
- remove all images
```bash
docker rmi -f $(docker images -aq)
```
