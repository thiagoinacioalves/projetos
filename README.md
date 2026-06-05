# Projetos

Repositório com projetos de infraestrutura, com foco atual em implantação no Kubernetes.

## Estrutura principal

```text
infra/
	README.md
	sonar-chart/
	sonarqube/
```

## Ambiente de infra

A documentação de infraestrutura está em `infra/README.md`.

## Projetos atuais do Sonar

- `infra/sonar-chart`: chart Helm do SonarQube + PostgreSQL.
- `infra/sonarqube`: manifests YAML e chart Helm espelhado para o mesmo stack.

Os detalhes técnicos de cada app estão dentro das respectivas pastas.
