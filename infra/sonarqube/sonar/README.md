# App SonarQube

Documentação detalhada da aplicação SonarQube via manifests YAML.

## Recursos da app

- `01-sonar-deploy.yaml`: Deployment do SonarQube
- `02-sonar-svc.yaml`: Service interno (porta 9000)
- `03-sonar-ingress.yaml`: Ingress para acesso externo
- `04-sonar-configmap-config.yaml`: configurações de runtime
- `04-sonar-configmap-copy.yaml`: scripts de cópia de plugins
- `04-sonar-configmap-install.yaml`: scripts de instalação de plugins
- `04-sonar-configmap-tests.yaml`: scripts de teste/validação
- `05-sonar-pvc.yaml`: volume persistente de dados
- `06-sonar-secret.yaml`: segredos (JDBC e credenciais)

## Ordem de deploy

```bash
kubectl apply -f 04-sonar-configmap-config.yaml -n sonarqube
kubectl apply -f 04-sonar-configmap-copy.yaml -n sonarqube
kubectl apply -f 04-sonar-configmap-install.yaml -n sonarqube
kubectl apply -f 04-sonar-configmap-tests.yaml -n sonarqube
kubectl apply -f 06-sonar-secret.yaml -n sonarqube
kubectl apply -f 05-sonar-pvc.yaml -n sonarqube
kubectl apply -f 01-sonar-deploy.yaml -n sonarqube
kubectl apply -f 02-sonar-svc.yaml -n sonarqube
kubectl apply -f 03-sonar-ingress.yaml -n sonarqube
```

## Ajustes comuns

### Versão do SonarQube
Editar imagem em `01-sonar-deploy.yaml`.

### Host do Ingress
Editar `spec.rules.host` em `03-sonar-ingress.yaml`.

### Recursos (CPU/Memória)
Editar bloco `resources` em `01-sonar-deploy.yaml`.

### Persistência
Editar capacidade em `05-sonar-pvc.yaml`.

## Validação

```bash
kubectl get deploy,svc,ingress,pvc -n sonarqube | grep sonar
kubectl logs -f deployment/sonarqube-sonarqube -n sonarqube
```
