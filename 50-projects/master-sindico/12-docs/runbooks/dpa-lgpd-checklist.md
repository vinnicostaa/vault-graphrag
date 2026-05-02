---
title: Runbook — DPA LGPD checklist (Cloudflare + AWS)
type: guide
tags:
  - runbook
  - dpa
  - lgpd
  - compliance
  - cloudflare
  - aws
  - master-sindico
  - v2
source: Vault §8-security/lgpd + AWS Artifact + Cloudflare DPA portal + ANPD guidance
created: '2026-04-27'
updated: '2026-04-27'
---

# Runbook — DPA LGPD checklist (Cloudflare + AWS)

**Bloqueador P0 pré-prod**. Sem ambas as DPAs assinadas, nenhum dado pessoal real pode entrar em prod (LGPD Art. 33 — operador requer DPA com controlador).

## 1. Cloudflare DPA

**Quem**: João (controlador) — só admin owner pode assinar.

**Procedure**:
1. Login dashboard https://dash.cloudflare.com
2. Account → Compliance → Data Processing Agreement
3. Ler integralmente (cobre GDPR + assemelhado LGPD; ANPD aceita)
4. Aceitar via clique + IP/timestamp registrado por CF
5. Download PDF assinado (auto-emitido)
6. Arquivar:
   - Drive corporativo: `Compliance/DPAs/cloudflare-dpa-<YYYY-MM-DD>.pdf`
   - Vault: entry [[../../STATE]] D-### "DPA Cloudflare assinada YYYY-MM-DD"
7. **Nota DPO**: registrar em registro de operações de tratamento (RIPD/RAT)

**SLA**: imediato (autoassinatura digital).

**Validar**: Cloudflare Compliance Page deve listar `Account Name` + data efetiva.

## 2. AWS DPA (GDPR Addendum)

**Quem**: João via root account ou IAM com `aws-portal:ViewAccount`.

**Procedure**:
1. Login AWS Console
2. Buscar serviço "AWS Artifact"
3. Agreements → Active Agreements → "AWS GDPR Data Processing Addendum"
4. Click "Accept agreement"
5. Confirmar identidade (typed name)
6. Download PDF
7. Arquivar:
   - Drive corporativo: `Compliance/DPAs/aws-gdpr-dpa-<YYYY-MM-DD>.pdf`
   - Vault: entry [[../../STATE]] D-### "DPA AWS assinada YYYY-MM-DD"

**SLA**: imediato.

**Validar**: AWS Artifact mostra status "Accepted by <user>" + timestamp.

## 3. SaaS externos — DPAs adicionais

| Provider | Onde | SLA | Status |
|---|---|---|---|
| Stripe (billing/PII pagamentos) | dashboard → Settings → Compliance → DPA | imediato | ⏳ |
| Mux (vídeos VOD com possível PII em metadata) | suporte enterprise; DPA standard via formulário | 5-10d | ⏳ |
| LiveKit (assembleia live com vídeo participantes) | dashboard → Settings → Compliance | imediato | ⏳ |
| Zitadel (identidade — controlador delegado) | suporte; DPA enterprise | 5-15d | ⏳ |
| Resend (email PII) | dashboard → Settings → Compliance | imediato | ⏳ |
| Twilio (SMS PII telefone) | console → Compliance | imediato | ⏳ |
| Sentry (logs com PII potencial — postergado M2 D-142) | suporte | 5d | ⏳ M2 |

## 4. Cláusulas críticas a verificar (todas DPAs)

- [ ] **Sub-processadores listados** (CF tem ~20; AWS tem ~50; Stripe tem ~10)
- [ ] **Direito do controlador auditar** (ANPD pode requisitar)
- [ ] **Notificação de breach em 72h** (ANPD Art. 48)
- [ ] **Direito DSAR** (acesso, exclusão, portabilidade)
- [ ] **Transferência internacional** com salvaguardas (CF/AWS armazenam globalmente)
- [ ] **Retorno/destruição de dados pós-término**

## 5. Cláusulas LGPD-específicas a anexar (se DPA é GDPR-only)

Algumas DPAs cobrem GDPR mas não citam LGPD. Anexo "LGPD Compliance Annex" que cobre:
- ANPD como autoridade competente
- LGPD Art. 33 (operador)
- LGPD Art. 48 (breach notification 72h)
- LGPD Art. 7 (bases legais — consentimento, contrato, obrigação legal, etc)
- LGPD Art. 18 (direitos do titular)

**Template**: pedir DPO redigir; anexar via assinatura digital secundária no Drive corporativo.

## 6. Registro de Operações (RIPD/RAT — LGPD Art. 37)

Para cada provider DPA assinada, criar entry no RIPD com:
- Finalidade do tratamento
- Categoria de dados pessoais (CPF, email, telefone, vídeo, geolocalização)
- Categorias de titulares (síndicos, moradores, empresas, etc)
- Tempo de retenção
- Provider (operador) + DPA reference
- Medidas de segurança aplicáveis

**Owner**: DPO. Ver vault [[../../08-security/lgpd]] §RIPD.

## 7. Audit trail no vault

Após cada DPA assinada, adicionar em [[../../STATE]]:

```markdown
## D-### — DPA <provider> assinada YYYY-MM-DD

- **Quem assinou**: João (controlador)
- **Data efetiva**: YYYY-MM-DD
- **PDF**: `Drive/Compliance/DPAs/<arquivo>.pdf` (SHA-256: ...)
- **Sub-processadores aceitos**: ver lista em PDF §X
- **Próxima revisão**: YYYY-MM-DD (anual)
- **Vault link**: [[../12-docs/runbooks/dpa-lgpd-checklist]]
```

## 8. Renovação anual

Marcar calendário. Providers podem atualizar DPA com novos sub-processadores; cliente reconfirma.

## 9. Anti-padrões

- ❌ Subir dado pessoal real em staging sem DPA assinada
- ❌ Usar provider sem DPA "porque é só dev" (LGPD não distingue)
- ❌ Não registrar DPA no RIPD
- ❌ Aceitar DPA sem ler (auditoria ANPD pode exigir comprovação de ciência)
- ❌ Manter PDF DPA só no email (Drive corporativo + vault entry obrigatório)

## Links

- [[_moc]]
- [[../../STATE]] (registro D-###)
- [[../../08-security/lgpd]]
- [[incident-lgpd-breach]]
- [[secret-rotation]]
- ANPD: https://www.gov.br/anpd/pt-br
- LGPD Art. 33 (operador): http://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm
- Cloudflare DPA: https://www.cloudflare.com/cloudflare-customer-dpa/
- AWS GDPR DPA: https://aws.amazon.com/compliance/gdpr-center/
- AWS Artifact: https://console.aws.amazon.com/artifact/
