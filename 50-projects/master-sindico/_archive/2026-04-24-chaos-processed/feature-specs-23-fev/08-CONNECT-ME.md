# 08 — CONNECT ME (Formulário de Contato)

> Sprint 1 · Rota: /connect-me/:userId (modal ou page)
> NÃO é chat. NÃO salva histórico. NÃO tem reply.

---

## REGRAS DE NEGÓCIO

- Formulário de contato UNIDIRECIONAL — disparo único de email/SMS
- Direções permitidas (Matriz Funcional):
  - Morador → Síndico (Base 2/ano, Pagante 4/ano)
  - Síndico → Empresa
  - Empresa Plus/Pro → Empresa (entre empresas)
- Empresas de Marketing NÃO têm Connect Me para abordar proativamente (anti-prospecção)
- O contato parte SEMPRE do interesse do contratante via busca
- Cota de uso validada pelo backend

## LAYOUT

```
┌──────────────────────────────────────────────┐
│  CONNECT ME                                   │
│  Entrar em contato com João Silva (Síndico)   │
│                                               │
│  ⚠️ 3 de 4 contatos disponíveis este ano     │
│  ████████████████████░░░░░ 75%                │
│                                               │
│  [Seu nome _________________________]        │
│  [Seu email ________________________]        │
│  [Telefone (opcional) ______________]        │
│                                               │
│  Assunto: [▼ Selecione categoria]            │
│                                               │
│  Mensagem:                                    │
│  ┌──────────────────────────────────────────┐│
│  │                                          ││
│  │                                   450/500││
│  └──────────────────────────────────────────┘│
│                                               │
│  ☑ Concordo que este formulário será enviado │
│    por email. Não há chat ou reply.           │
│                                               │
│  [Enviar Formulário →] primary               │
│                                               │
│  ℹ️ O contato será feito fora da plataforma  │
│  pelo destinatário.                           │
└──────────────────────────────────────────────┘
```

## CSS

```css
.connect-me-quota {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 16px; background: var(--warning) / 0.08;
  border: 1px solid var(--warning) / 0.2; border-radius: var(--radius);
  font-family: 'Manrope'; font-size: 13px; color: var(--foreground);
  margin-bottom: 24px;
}
.connect-me-quota-bar { flex: 1; height: 4px; background: var(--muted); border-radius: 2px; }
.connect-me-quota-fill { height: 100%; background: var(--warning); border-radius: 2px; }
.connect-me-disclaimer {
  font-size: 12px; color: var(--muted-foreground); display: flex; gap: 8px;
  padding: 12px; background: var(--muted); border-radius: var(--radius);
  margin-top: 16px;
}
```

## ESTADOS

| Estado | UI |
|---|---|
| Cota disponível | Formulário ativo, barra de cota visível |
| Cota esgotada | Formulário disabled, mensagem "Limite anual atingido" |
| Enviando | Botão spinner + "Enviando..." |
| Enviado | Toast success + redirect para perfil do destinatário |
| Erro | Toast destructive + form permanece |

## COMPONENTES

ConnectMeForm, QuotaBar, ConnectMeDisclaimer
