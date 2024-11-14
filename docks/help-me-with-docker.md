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
