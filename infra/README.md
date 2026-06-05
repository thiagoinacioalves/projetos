# Infraestrutura

Este diretório concentra os artefatos de infraestrutura do ambiente SonarQube.

## Projetos atuais

### 1) `sonar-chart`
Chart Helm para implantação do SonarQube com PostgreSQL.

### 2) `sonarqube`
Estrutura com duas abordagens:
- `sonar-chart/`: chart Helm
- `sonar/` e `postgres-sonar/`: manifests YAML separados por app

## Onde está a descrição detalhada

A documentação detalhada de cada app fica dentro das próprias pastas:

- `infra/sonarqube/sonar/README.md`
- `infra/sonarqube/postgres-sonar/README.md`
- `infra/sonarqube/sonar-chart/README.md`

## Uso rápido

### Helm
```bash
cd infra/sonarqube
helm install sonarqube ./sonar-chart -n sonarqube --create-namespace
```

### Manifests YAML
```bash
kubectl apply -f infra/sonarqube/postgres-sonar/ -n sonarqube
kubectl apply -f infra/sonarqube/sonar/ -n sonarqube
```
