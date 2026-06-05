# App PostgreSQL (Sonar)

Documentação detalhada da aplicação PostgreSQL que suporta o SonarQube.

## Recursos da app

- `01-postgres-stateful.yaml`: StatefulSet do PostgreSQL
- `02-postgres-svc.yaml`: Service principal
- `03-postgres-svc-headless.yaml`: Service headless para StatefulSet
- `04-sonar-configmap-config.yaml`: configuração do banco
- `04-postgres-secret.yaml`: credenciais do PostgreSQL
- `05-postgres-pvc.yaml`: volume persistente

## Ordem de deploy

```bash
kubectl apply -f 04-sonar-configmap-config.yaml -n sonarqube
kubectl apply -f 04-postgres-secret.yaml -n sonarqube
kubectl apply -f 05-postgres-pvc.yaml -n sonarqube
kubectl apply -f 03-postgres-svc-headless.yaml -n sonarqube
kubectl apply -f 02-postgres-svc.yaml -n sonarqube
kubectl apply -f 01-postgres-stateful.yaml -n sonarqube
```

## Ajustes comuns

### Versão do PostgreSQL
Editar imagem no `01-postgres-stateful.yaml`.

### Tamanho do volume
Editar `resources.requests.storage` em `05-postgres-pvc.yaml`.

### Senha do banco
Atualizar `04-postgres-secret.yaml` com novo valor em base64.

## Validação

```bash
kubectl get sts,svc,pvc -n sonarqube | grep postgres
kubectl logs -f statefulset/sonarqube-db-postgresql -n sonarqube
kubectl exec -it sonarqube-db-postgresql-0 -n sonarqube -- psql -U sonar -d sonar
```
