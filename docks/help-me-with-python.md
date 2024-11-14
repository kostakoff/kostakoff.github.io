# Instruction

## Requirments
- [X] python 3.9 ot higher
- [X] virtualenv

## virtualenv preparation
- create virtualenv for python3
```bash
virtualenv --python=python3 ~/.venv3
```
- load virtualenv
```bash
source ~/.venv3/bin/activate
```

## python hot commands
- install PDM package manager
```bash
pip install pdm
```
- PDM enable immuteble pipeline libraries (execute this command for each new terminal session)
```bash
eval "$(pdm --pep582)"
```
