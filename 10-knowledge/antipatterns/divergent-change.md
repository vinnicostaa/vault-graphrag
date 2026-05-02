---
title: Divergent Change (AP-022)
type: antipattern
tags: [antipattern, code-smell, change-preventer, cohesion]
source: https://refactoring.guru/smells/divergent-change
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [low-cohesion-class, too-many-reasons-to-change]
---

# AP-022 — Divergent Change

Uma **única classe** muda por **muitas razões diferentes**. Toda feature de A, B, C mexe no mesmo arquivo — mesmo quando A, B, C são temas independentes. Viola SRP de forma ostensiva. Smell oposto, porém simétrico, do [[shotgun-surgery]].

## Sintomas

- Arquivo `UserService.java` mudou 50× em 6 meses, com razões completamente diferentes (auth, billing, notification, report)
- Git blame mostra 15 autores distintos cada um com seu motivo
- PR review aponta "isso também deveria mudar outros lugares?" recorrentemente
- Testes da classe são organizados por seções `// billing tests`, `// auth tests`, `// export tests`
- Merge conflicts constantes entre squads que mexem na mesma classe por razões ortogonais
- Injeção de 10+ dependências no construtor

## Por que é problema

**Baixa coesão**: classe mistura responsabilidades que não compartilham ritmo de mudança. Cada feature paga custo de **não quebrar as outras**.

**Blast radius grande**: mudar 1 razão pode derrubar outras N razões do mesmo arquivo.

**Deploy gargalo**: squads diferentes não conseguem evoluir em paralelo.

**Conflitos e bugs cross-feature**: auth muda e billing quebra sem relação causal aparente.

**SRP violada**: "uma classe deve ter uma única razão pra mudar" (Martin). Se tem N, é N classes mal-posicionadas.

## Divergent Change vs Shotgun Surgery

Eles são o **espelho** um do outro:

| Smell | O que acontece |
|---|---|
| **Divergent Change** | 1 mudança no produto → N edições **em 1 classe** (classe sabe demais) |
| **Shotgun Surgery** | 1 mudança no produto → 1 edição **em N classes** (conceito espalhado) |

Os dois violam coesão — mas em direções opostas. Fix de um pode introduzir o outro se feito sem critério.

## Exemplo

```python
# ❌ Divergent Change — UserService muda por 4 razões
class UserService:
    def authenticate(self, email, password): ...     # razão 1: auth
    def send_invoice(self, user_id, items): ...      # razão 2: billing
    def notify_campaign(self, segment): ...          # razão 3: marketing
    def export_lgpd(self, user_id): ...              # razão 4: compliance
# cada feature mexe aqui e ninguém trabalha em paralelo sem conflito

# ✅ Extrair por razão de mudança
class AuthService: ...              # muda quando política de auth muda
class InvoiceService: ...           # muda quando billing muda
class CampaignService: ...          # muda quando marketing muda
class LgpdExportService: ...        # muda quando compliance muda
```

## Como refatorar

- **Extract Class**: separar por razão-de-mudança (cohesion axis). Perguntar "se X muda, o que precisa mudar junto?"
- **Move Method / Move Field**: alinhar métodos com a classe que compartilha o ritmo de mudança
- **Introduce Parameter Object / Value Object**: extrair conceitos que andam juntos
- **Bounded Context split** (DDD): às vezes a classe é gigante porque abriga **vários contextos** — separar por contexto é o refactor real
- **Avaliar git log**: `git log --oneline --stat <arquivo>` agrupando por tipo de mudança revela os eixos naturais

## Quando tolerar

- **Facade / Aggregate root** que **intencionalmente** coordena múltiplos colaboradores — mas ela deve delegar, não implementar cada razão
- **Domain Service** que atravessa conceitos por natureza (ex: `PricingService` que vê desconto + tax + currency) — legítimo se é **o domínio** que é assim
- **Entidade raiz** em DDD com invariantes que cruzam aspectos — é o papel dela
- **Fase inicial do projeto** — antes de coesão emergir, classe única pode ser ok; quando ritmos divergirem, partir

## Patterns relacionados (que resolvem)

- [[shotgun-surgery]] — smell espelho; cuidado pra não cair nele ao refatorar
- [[god-object]] — Divergent Change é frequentemente sintoma de God Object
- [[../patterns/strategy]] — extrair variação por razão-de-mudança em estratégias
- [[../patterns/facade]] — coordenar sem implementar cada razão
- [[../architecture/ddd-strategic-tactical]] — bounded contexts como eixos de mudança
- [[../principles/solid]] — SRP é a regra direta

## Fontes

- Refactoring Guru — [Divergent Change](https://refactoring.guru/smells/divergent-change)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2017). *Clean Architecture*. Prentice Hall — SRP como "axis of change".

## Links

- [[_moc]]
- [[../_moc]]
- [[shotgun-surgery]]
- [[god-object]]
- [[../principles/solid]]
