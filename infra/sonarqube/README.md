# SonarQube na Infra

Esta pasta reúne os projetos de implantação do SonarQube no Kubernetes.

## Projetos atuais

- `sonar/`: app SonarQube (Deployment, Service, Ingress, ConfigMaps, Secret, PVC)
- `postgres-sonar/`: app PostgreSQL (StatefulSet, Services, ConfigMap, Secret, PVC)
- `sonar-chart/`: opção Helm para subir todo o stack

## Fluxo recomendado

1. Aplicar banco (`postgres-sonar`)
2. Validar readiness do PostgreSQL
3. Aplicar aplicação (`sonar`)

## Instalação rápida (YAML)

```bash
kubectl apply -f postgres-sonar/ -n sonarqube
kubectl wait --for=condition=ready pod -l app=postgresql -n sonarqube --timeout=300s
kubectl apply -f sonar/ -n sonarqube
```

## Instalação rápida (Helm)

```bash
helm install sonarqube ./sonar-chart -n sonarqube --create-namespace
```

## Documentação detalhada por app

- `sonar/README.md`
- `postgres-sonar/README.md`
- `sonar-chart/README.md`
