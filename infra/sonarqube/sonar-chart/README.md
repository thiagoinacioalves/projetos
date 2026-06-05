# SonarQube - Guia de Implantação

Este repositório contém duas formas de implantar o SonarQube com PostgreSQL no Kubernetes:

1. **Helm Chart** (Recomendado) - Implantação parametrizada e reutilizável
2. **Manifests YAML** - Implantação direta com kubectl

---

## 📦 Opção 1: Helm Chart (Recomendado)

### Por que usar Helm?

- ✅ Parametrização completa via `values.yaml`
- ✅ Fácil atualização e rollback
- ✅ Reutilizável em múltiplos ambientes
- ✅ Controle granular dos componentes (habilitar/desabilitar initContainers)
- ✅ Versionamento do chart

### Pré-requisitos

- Kubernetes 1.19+
- Helm 3.0+
- StorageClass configurada (padrão: nfs-client)

### Instalação

```bash
# Instalar o chart
helm install sonarqube ./sonar-chart

# Instalar com valores customizados
helm install sonarqube ./sonar-chart -f custom-values.yaml

# Instalar em um namespace específico
helm install sonarqube ./sonar-chart -n sonarqube --create-namespace
```

## Desinstalação

```bash
helm uninstall sonarqube
```

## Configuração

Os seguintes parâmetros podem ser configurados no arquivo `values.yaml`:

### SonarQube

| Parâmetro | Descrição | Valor Padrão |
|-----------|-----------|--------------|
| `sonarqube.enabled` | Habilita o SonarQube | `true` |
| `sonarqube.image.repository` | Repositório da imagem | `sonarqube` |
| `sonarqube.image.tag` | Tag da imagem | `26.4.0.121862-community` |
| `sonarqube.replicas` | Número de réplicas | `1` |
| `sonarqube.service.port` | Porta do serviço | `9000` |
| `sonarqube.ingress.enabled` | Habilita Ingress | `true` |
| `sonarqube.ingress.host` | Host do Ingress | `sonarqube.joinhati.com.br` |
| `sonarqube.persistence.size` | Tamanho do PVC | `10Gi` |
| `sonarqube.persistence.storageClass` | StorageClass | `nfs-client` |
| `sonarqube.initContainers.initVmMaxMapCount.enabled` | Habilita initContainer vm.max_map_count | `true` |
| `sonarqube.initContainers.downloadPlugins.enabled` | Habilita initContainer de download de plugins | `true` |

### PostgreSQL

| Parâmetro | Descrição | Valor Padrão |
|-----------|-----------|--------------|
| `postgresql.enabled` | Habilita o PostgreSQL | `true` |
| `postgresql.image.repository` | Repositório da imagem | `postgres` |
| `postgresql.image.tag` | Tag da imagem | `16.1` |
| `postgresql.replicas` | Número de réplicas | `1` |
| `postgresql.service.port` | Porta do serviço | `5432` |
| `postgresql.persistence.size` | Tamanho do PVC | `30Gi` |
| `postgresql.persistence.storageClass` | StorageClass | `nfs-client` |
| `postgresql.initContainers.fixPermissions.enabled` | Habilita initContainer fix-permissions | `true` |

## Exemplo de valores customizados

```yaml
sonarqube:
  ingress:
    host: meu-sonarqube.exemplo.com
  persistence:
    size: 20Gi
  resources:
    limits:
      cpu: "2"
      memory: 8192Mi
  # Desabilitar initContainers se necessário
  initContainers:
    initVmMaxMapCount:
      enabled: false
    downloadPlugins:
      enabled: true

postgresql:
  persistence:
    size: 50Gi
  resources:
    limits:
      memory: 8Gi
  initContainers:
    fixPermissions:
      enabled: true
```

## Acesso

Após a instalação, o SonarQube estará disponível em:
- URL: http://sonarqube.joinhati.com.br (ou o host configurado)
- Credenciais padrão: admin/admin (altere na primeira login)

## Observações

- O chart inclui o plugin community-branch-plugin para suporte a branches
- As senhas estão em base64 no values.yaml (altere antes da instalação em produção)
- O PostgreSQL usa um StatefulSet para garantir persistência
- InitContainers configuram vm.max_map_count e baixam plugins

---

## 📄 Opção 2: Manifests YAML

### Por que usar Manifests YAML?

- ✅ Controle direto sobre cada recurso
- ✅ Ideal para GitOps (ArgoCD, Flux)
- ✅ Sem dependências externas (apenas kubectl)
- ✅ Fácil de versionar no Git

### Estrutura dos Manifests

```
infra/sonarqube/
├── sonar/
│   ├── 01-sonar-deploy.yaml              # Deployment do SonarQube
│   ├── 02-sonar-svc.yaml                 # Service do SonarQube
│   ├── 03-sonar-ingress.yaml             # Ingress
│   ├── 04-sonar-configmap-config.yaml    # ConfigMap de configuração
│   ├── 04-sonar-configmap-copy.yaml      # ConfigMap para cópia de plugins
│   ├── 04-sonar-configmap-install.yaml   # ConfigMap para instalação
│   ├── 04-sonar-configmap-tests.yaml     # ConfigMap de testes
│   ├── 05-sonar-pvc.yaml                 # PersistentVolumeClaim
│   └── 06-sonar-secret.yaml              # Secret com senhas
└── postgres-sonar/
    ├── 01-postgres-stateful.yaml         # StatefulSet do PostgreSQL
    ├── 02-postgres-svc.yaml              # Service principal
    ├── 03-postgres-svc-headless.yaml     # Service headless
    ├── 04-sonar-configmap-config.yaml    # ConfigMap do PostgreSQL
    ├── 04-postgres-secret.yaml           # Secret com senhas
    └── 05-postgres-pvc.yaml              # PersistentVolumeClaim
```

### Instalação com Kubectl

```bash
# Criar namespace (opcional)
kubectl create namespace sonarqube

# Aplicar PostgreSQL
kubectl apply -f infra/sonarqube/postgres-sonar/ -n sonarqube

# Aguardar PostgreSQL estar pronto
kubectl wait --for=condition=ready pod -l app=postgresql -n sonarqube --timeout=300s

# Aplicar SonarQube
kubectl apply -f infra/sonarqube/sonar/ -n sonarqube

# Verificar status
kubectl get pods -n sonarqube
```

### Desinstalação com Kubectl

```bash
# Remover recursos
kubectl delete -f infra/sonarqube/sonar/ -n sonarqube
kubectl delete -f infra/sonarqube/postgres-sonar/ -n sonarqube

# Remover PVCs (se necessário)
kubectl delete pvc sonarqube-pvc sonarqube-db-postgresql-pvc -n sonarqube
```

### Personalização dos Manifests

Para customizar a implantação, edite os arquivos YAML diretamente:

#### Alterar versão do SonarQube:
```yaml
# infra/sonarqube/sonar/01-sonar-deploy.yaml
spec:
  template:
    spec:
      containers:
        - name: sonarqube
          image: sonarqube:26.4.0.121862-community  # Altere aqui
```

#### Alterar host do Ingress:
```yaml
# infra/sonarqube/sonar/03-sonar-ingress.yaml
spec:
  rules:
    - host: sonarqube.joinhati.com.br  # Altere aqui
```

#### Alterar senhas (base64):
```bash
# Gerar nova senha em base64
echo -n "nova-senha" | base64

# Editar secrets
# infra/sonarqube/sonar/06-sonar-secret.yaml
# infra/sonarqube/postgres-sonar/04-postgres-secret.yaml
```

#### Alterar recursos (CPU/Memory):
```yaml
# infra/sonarqube/sonar/01-sonar-deploy.yaml
resources:
  limits:
    cpu: "2"           # Altere aqui
    memory: 8192Mi     # Altere aqui
  requests:
    cpu: 500m
    memory: 1024Mi
```

#### Alterar tamanho do storage:
```yaml
# infra/sonarqube/sonar/05-sonar-pvc.yaml
resources:
  requests:
    storage: 20Gi  # Altere aqui

# infra/sonarqube/postgres-sonar/05-postgres-pvc.yaml
resources:
  requests:
    storage: 50Gi  # Altere aqui
```

### Uso com GitOps (ArgoCD)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sonarqube
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://seu-repo-git.com/gitops
    targetRevision: main
    path: infra/sonarqube
  destination:
    server: https://kubernetes.default.svc
    namespace: sonarqube
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

---

## 🔄 Comparação: Helm vs YAML

| Aspecto | Helm Chart | Manifests YAML |
|---------|------------|----------------|
| **Facilidade de instalação** | ⭐⭐⭐⭐⭐ Um comando | ⭐⭐⭐ Múltiplos comandos |
| **Parametrização** | ⭐⭐⭐⭐⭐ values.yaml | ⭐⭐⭐ Edição manual |
| **Reutilização** | ⭐⭐⭐⭐⭐ Multi-ambientes | ⭐⭐⭐ Requer duplicação |
| **Rollback** | ⭐⭐⭐⭐⭐ `helm rollback` | ⭐⭐ Manual via Git |
| **GitOps** | ⭐⭐⭐⭐ Suportado | ⭐⭐⭐⭐⭐ Nativo |
| **Versionamento** | ⭐⭐⭐⭐⭐ Chart version | ⭐⭐⭐⭐ Git commits |
| **Complexidade** | ⭐⭐⭐ Templates | ⭐⭐⭐⭐⭐ Simples |
| **Debugging** | ⭐⭐⭐ helm template | ⭐⭐⭐⭐⭐ Direto |

### Quando usar Helm?
- Múltiplos ambientes (dev, staging, prod)
- Precisa de parametrização frequente
- Equipe familiarizada com Helm
- Quer facilitar upgrades e rollbacks

### Quando usar YAML?
- Pipeline GitOps estabelecido
- Configuração estática
- Controle granular necessário
- Evitar dependência do Helm

---

## 🔐 Acesso

Após a instalação (qualquer método), o SonarQube estará disponível em:
- **URL**: http://sonarqube.joinhati.com.br (ou o host configurado)
- **Credenciais padrão**: `admin` / `admin` (altere no primeiro login)

---

## 📝 Notas Importantes

- **vm.max_map_count**: Os initContainers configuram automaticamente (requer privilégios)
- **Plugin Community Branch**: Baixado automaticamente na versão 26.4.0
- **Senhas**: Todas em base64 - **ALTERE EM PRODUÇÃO**
- **Storage**: Certifique-se que a StorageClass `nfs-client` existe ou ajuste
- **Recursos**: Valores padrão são para ambientes de desenvolvimento, ajuste para produção

---

## 🆘 Troubleshooting

### SonarQube não inicia

```bash
# Verificar logs
kubectl logs -f deployment/sonarqube-sonarqube -n sonarqube

# Verificar se PostgreSQL está pronto
kubectl get pods -l app=postgresql -n sonarqube

# Verificar vm.max_map_count
kubectl exec -it deployment/sonarqube-sonarqube -n sonarqube -- sysctl vm.max_map_count
```

### Erro de conexão com PostgreSQL

```bash
# Verificar secret
kubectl get secret sonarqube-secret -n sonarqube -o yaml

# Testar conexão
kubectl exec -it deployment/sonarqube-sonarqube -n sonarqube -- \
  psql -h sonarqube-db-postgresql -U sonar -d sonar
```

### Problemas com plugins

```bash
# Verificar se plugins foram baixados
kubectl exec -it deployment/sonarqube-sonarqube -n sonarqube -- \
  ls -la /opt/sonarqube/extensions/plugins/

# Recriar pods para forçar download
kubectl rollout restart deployment/sonarqube-sonarqube -n sonarqube
```

---

## 📚 Recursos Adicionais

- [Documentação SonarQube](https://docs.sonarqube.org/)
- [Community Branch Plugin](https://github.com/mc1arke/sonarqube-community-branch-plugin)
- [PostgreSQL no Kubernetes](https://www.postgresql.org/docs/current/index.html)
- [Helm Documentation](https://helm.sh/docs/)

---

## 🤝 Contribuindo

Para contribuir com melhorias:
1. Faça um fork do repositório
2. Crie uma branch para sua feature
3. Commit suas mudanças
4. Abra um Pull Request

---

## 📄 Licença

Este projeto é baseado em componentes open source. Consulte as licenças individuais:
- SonarQube: LGPL-3.0
- PostgreSQL: PostgreSQL License
- Community Branch Plugin: LGPL-3.0
