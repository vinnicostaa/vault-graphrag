---
title: Netflix Spinnaker — CD em escala (pipelines, canary, blue/green)
type: source
tags: [spinnaker, continuous-delivery, deployment, canary, blue-green, netflix, research]
source: https://spinnaker.io/
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-spinnaker]
---

# Netflix Spinnaker: CD em escala

> Fontes: spinnaker.io, Netflix TechBlog "Global CD with Spinnaker", CD Foundation (Temporal integration, fev/2026), The New Stack — destilado 2026-04-23.

## O que é

Multi-cloud continuous delivery platform, criado por Netflix + Google em 2014-2015. Hoje orquestra **> 95% dos deploys Netflix em AWS**, milhares de deploys/dia.

## Conceitos

- **Application** → **Pipeline** → **Stage** → **Task**.
- Pipelines encadeiam stages serial/paralelo, triggers por git push, Jenkins, Docker registry push, CRON, outro pipeline.
- **Deployment strategies nativas**:
  - **Red/Black (Blue/Green)** — cluster novo paralelo, shift de tráfego, drop do antigo se ok.
  - **Canary** — rollout gradual (1% → 10% → 50% → 100%) com análise automática de métricas (Kayenta).
  - **Rolling** — substituição incremental in-place.
- Integrações: AWS, GCP, Azure, Kubernetes, OCI, Cloud Foundry, OpenStack.

## Atualização 2024-2026

- **Temporal** agora orquestra cloud operations do Spinnaker (resolve 4% de deploy failures causados por falhas transientes).
- **Kubernetes plugin v2** usa `kubectl` nativo + suporta CRDs.
- RBAC: OAuth/SAML/LDAP/Google Groups.
- Integração Chaos Monkey nativa (canary + chaos na mesma pipeline).

## Princípios aplicáveis sem adotar Spinnaker

1. **Pipeline as code** — deploy não pode ser passo manual.
2. **Immutable infrastructure** — imagens assadas (Packer), não muta em prod.
3. **Automated canary analysis** — métricas comparadas entre canary e baseline automaticamente; rollback se regressão.
4. **Manual judgments com RBAC** — aprovação humana em gates críticos (prod deploy).
5. **Restricted execution windows** — deploy só em janela definida (ex: nunca sexta 18h).

## Quando Spinnaker faz sentido

- Múltiplos clusters, múltiplas regiões, centenas de serviços, dezenas de deploys/dia.
- Time dedicado a plataforma (Spinnaker operacionalmente complexo).

## Quando NÃO

- 1-5 serviços, 1 cluster, deploy por semana → Spinnaker é overkill.

## Aplicabilidade ao Master Síndico

**NÃO adotar Spinnaker.** Stack mais simples cobre:

### M1-M2: GitHub Actions ou GitLab CI

- Pipeline em YAML versionado.
- Build → test → deploy → smoke test.
- Blue/green via infra (Kubernetes deployment + service switch) ou flag (Argo Rollouts se k8s).

### M3+: Argo CD + Argo Rollouts

- GitOps-native, Kubernetes-first, muito mais leve que Spinnaker.
- Argo Rollouts cobre canary e blue/green.
- Kayenta (automated canary analysis) pode ser usado standalone.

### Princípios a aplicar DESDE JÁ

- [ ] Immutable container images (tag = commit SHA, nunca `:latest`).
- [ ] Canary automático com rollback em regressão de SLO.
- [ ] Janelas de deploy definidas.
- [ ] Approval gate em prod (mesmo se automatizado depois).
- [ ] Deploy pipeline fail-closed — se smoke test falha, rollback automático.

## Links

- [[source-05]] — chaos integrado com canary (Spinnaker + Chaos Monkey)
- [[source-10]] — observability pra canary analysis
- [[_destilado]]
