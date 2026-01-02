# Infraestrutura Kubernetes - Projeto Fiap 🔧

## Visão Geral
Este repositório contém os manifests de Kubernetes para provisionar a infraestrutura e os microserviços do projeto.
O objetivo do README é explicar a função de cada pasta/arquivo e fornecer um passo-a-passo para subir tudo em um cluster Kubernetes.

---

## Estrutura do repositório 📁

- `get_helm.sh` - Script utilitário para instalar/atualizar o Helm (se necessário).
- `k8s/` - Diretório principal com os manifests de Kubernetes.
  - `infra/`
    - `elasticsearch.yml` - Deployment/Service para Elasticsearch (armazenamento e busca).
    - `ingress.yml` - Regras de Ingress para expor serviços externamente (hostname, TLS, paths).
  - `monitoring/`
    - `grafana/`
      - `config-map.yml` - Configuração do Grafana (dashboards, datasources padrão).
      - `deployment.yml` - Deployment do Grafana.
      - `service.yml` - Service (ClusterIP/LoadBalancer) para Grafana.
    - `prometheus/`
      - `config-map.yml` - Configuração do Prometheus (scrape configs, regras).
      - `deployment.yml` - Deployment do Prometheus.
      - `rbac.yml` - Roles e RoleBindings necessários para Prometheus.
      - `service.yml` - Service para expor Prometheus.
  - `ms-auth-manager/`, `ms-games/`, `ms-payment/`, `ms-user/` (cada microserviço)
    - `deployment.yml` - Deployment do microserviço.
    - `hpa.yml` - Horizontal Pod Autoscaler (se configurado).
    - `secrets-template.yml` - Template com chaves necessárias (não com valores secretos reais).
    - `secrets.yml` - Arquivo de secrets (se presente, contém valores — **mantenha seguro/criptografado**).
    - `service.yml` - Service que expõe o microserviço internamente ou externamente.

---

## Pré-requisitos ✅

- Acesso a um cluster Kubernetes (kubectl configurado apontando para o cluster correto).
- `kubectl` instalado (v1.20+ recomendado).
- (Opcional) `helm` se preferir usar charts em vez de manifests diretos.
- Para HPA: `metrics-server` ou outro provedor de métricas instalado no cluster.
- Para TLS/Ingress: DNS apontando para o Ingress Controller e um controller (nginx/traefik) instalado.

---

## Como subir a infraestrutura (passo-a-passo) 🚀

1. (Opcional) Crie namespaces para isolar recursos:

```bash
kubectl create namespace infra
kubectl create namespace monitoring
kubectl create namespace apps
```

2. Aplicar infraestrutura básica (Elasticsearch e Ingress):

```bash
kubectl apply -f k8s/infra/elasticsearch.yml -n infra
kubectl apply -f k8s/infra/ingress.yml -n infra
```

> Observação: verifique se o Ingress Controller está instalado no cluster antes de aplicar o Ingress. Ajuste hostnames/TLS conforme necessário.

3. Deploy do monitoring (Prometheus e Grafana):

```bash
kubectl apply -f k8s/monitoring/prometheus -n monitoring
kubectl apply -f k8s/monitoring/grafana -n monitoring
```

- Acesse Grafana (ex.: `kubectl port-forward svc/grafana 3000:3000 -n monitoring`) e depois abra `http://localhost:3000`. Usuário/senha dependem da configuração (checar `config-map.yml` ou secrets).

4. Preparar e aplicar secrets dos microserviços

- Preencha `secrets-template.yml` com os valores reais e gere `secrets.yml` (ou use `kubectl create secret`):

```bash
kubectl create secret generic ms-auth-manager-secrets --from-literal=KEY=VALUE -n apps
# ou aplicar diretamente, se você já tiver o arquivo
kubectl apply -f k8s/ms-auth-manager/secrets.yml -n apps
```

> Recomenda-se usar soluções como Sealed Secrets, External Secrets ou vault para não manter segredos em texto puro no repositório.

5. Aplicar deployments e services dos microserviços:

```bash
kubectl apply -f k8s/ms-auth-manager -n apps
kubectl apply -f k8s/ms-games -n apps
kubectl apply -f k8s/ms-payment -n apps
kubectl apply -f k8s/ms-user -n apps
```

6. (Se aplicável) Habilitar HPA:

```bash
kubectl apply -f k8s/ms-auth-manager/hpa.yml -n apps
# Repita para outros microserviços com HPA
```

7. Verificações e troubleshooting:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace>
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

---

## Boas práticas & notas ⚠️

- Nunca comitar secrets em texto plano no repositório público.
- Considere usar CI/CD para builds e deploys (GitHub Actions, GitLab CI, etc.).
- Mantenha monitoramento e alertas configurados no Prometheus/Grafana.
- Teste o Ingress e TLS em um ambiente de staging antes de produção.

---

## Exemplo rápido de deploy (resumo)

1. Criar namespaces
2. Deploy infra: `k8s/infra`
3. Deploy monitoring: `k8s/monitoring`
4. Criar secrets dos microserviços
5. Deploy microserviços: `k8s/ms-*`
6. Aplica HPAs

---

## Ajuda / Próximos passos 💡

Se quiser, posso:
- Gerar um `kustomization.yaml` ou `Helm Chart` para facilitar deploys.
- Adicionar exemplos de CI/CD (pipeline) para deploy automático.
- Criar instruções em inglês ou detalhes mais específicos (TLS, DNS, storage classes).

---

**Pronto para rodar**: use os comandos acima adaptando namespaces, contextos e valores sensíveis ao seu ambiente.
