# Instruction

## Kubernetes hot commands

- uninstall all helm charts in namespace
```bash
helm uninstall $(helm ls --short)
```
