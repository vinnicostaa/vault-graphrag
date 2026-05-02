---
title: Test Traceability — Matriz Requirement → Test ID
type: spec
tags: [testing, traceability, coverage, requirements, ears, master-sindico, v2]
source: 04-requirements/{global,functional/*,matrix-functional}.md + 01-domain/{invariants,state-machines}.md + 09-testing/{unit,integration,e2e,pbt}.md
created: 2026-04-23
updated: 2026-04-23
aliases: [test-coverage-matrix, req-to-test, traceability-matrix]
---

# Test Traceability — Matriz Requirement → Test ID

Fechamento do finding **A-DC-QA-001** (🔴 QA). Matriz canônica **req→test_id** cobrindo os **246 requirements** (GR, IDN, BIL, INS, COM, CNT, ASM, XD) + **100 invariantes** (INV-001..INV-100) + **features** da matriz funcional + **transições** de state machines.

> Fonte da verdade para o gate de cobertura end-to-end. Toda Pull Request que introduz novo requirement **deve** adicionar linha aqui antes do merge (DoD §3).

---

## 1. Convenções

### 1.1 Formato do test_id

`T-<BC>-<REQ>-<tipo>-<slug-curto>`

- `<BC>`: `GR | IDN | BIL | INS | COM | CNT | ASM | XD` (alinhado com prefixo do requirement).
- `<REQ>`: número do requirement (ex: `010`, `042`).
- `<tipo>`: `unit | int | e2e | pbt` (✗ `pbt` só obrigatório pros 5 INV PBT).
- `<slug-curto>`: 2-5 palavras kebab-case descritivas da propriedade testada.

**Exemplos**:
- `T-IDN-010-int-device-invalidation`
- `T-BIL-016-pbt-quota-exhaustion`
- `T-ASM-074-unit-edital-required-before-start`

### 1.2 Tipos de teste (pirâmide 70-20-10)

| Tipo | Arquivo canônico | Quando usar |
|---|---|---|
| `unit` | `09-testing/unit.md` + `/tests/<bc>/<req>_test.go` | Regra de domínio (VO, factory, state transition simples) |
| `int` | `09-testing/integration.md` + `/tests/integration/<bc>/<req>_test.go` (`//go:build integration`) | Repo/UoW/tx/cross-tenant/webhook idempotência |
| `e2e` | `09-testing/e2e.md` + `/tests/e2e/<bc>/<req>.spec.ts` | Fluxo completo persona→API→DB |
| `pbt` | `09-testing/pbt.md` + `/tests/property/<bc>/<req>_property_test.go` | Combinatória/TOCTOU/matemática |

### 1.3 Heurística default (quando finding não aponta tipo explícito)

Aplicada nesta matriz quando o requirement não explicita teste:

| Camada do requirement | Tipo default | Arquivo default |
|---|---|---|
| domain (INV-### puros, VOs, factories) | `unit` | `/tests/<bc>/<slug>_test.go` |
| application (use cases, saga, UoW) | `unit + int` | dois test_ids |
| infrastructure (repo, provider, cron) | `int` | `/tests/integration/<bc>/<slug>_test.go` |
| http / webhook / rota pública | `e2e + int` (HMAC + idempotência) | `/tests/e2e/<bc>/<slug>.spec.ts` + int |
| cross-tenant / ABAC / LGPD | `int` (100+ casos sistemáticos) | `/tests/integration/abac/<slug>_test.go` |

### 1.4 Status de cobertura

- ✅ **Covered** — test_id existe em `unit.md`/`integration.md`/`e2e.md`/`pbt.md` com seção dedicada.
- 🟡 **Planned** — test_id previsto nesta matriz; arquivo de teste a criar durante Fase 6/M1.
- ⚠️ **Pendente** — requirement tem `⚠️ PENDÊNCIA` upstream; teste depende de reconciliação (ex: A-DC-QA-071).
- ⏭️ **Deferred** — marcado `W` (backlog) ou `C` M3+; teste fora do M1.

---

## 2. Global Requirements (GR-001..GR-035)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| GR-001 | int | `TestAuthMiddleware_when_zitadel_valid_should_set_claims` | `/tests/integration/identity/auth_middleware_test.go` | 🟡 Planned |
| GR-002 | pbt + int | `TestABACEngine_when_matrix_eval_should_deny_default` / `abac_engine_property_test.go` | `/tests/property/abac/engine_property_test.go` | ✅ Covered (INV-086 — 400 casos) |
| GR-003 | int | `TestTenantIsolation_when_cross_tenant_should_404` | `/tests/integration/abac/tenant_isolation_test.go` | ✅ Covered |
| GR-004 | int | `TestCORS_when_wildcard_in_prod_should_fail_startup` | `/tests/integration/middleware/cors_test.go` | 🟡 Planned |
| GR-005 | int | `TestRateLimiter_when_20rpm_burst5_should_throttle` | `/tests/integration/middleware/ratelimit_test.go` | 🟡 Planned |
| GR-006 | int | `TestCSRF_when_state_param_mismatch_should_fail` | `/tests/integration/auth/csrf_test.go` | 🟡 Planned |
| GR-007 | int | `TestSession_when_cookie_httpOnly_secure_samesite_should_match_INV100` | `/tests/integration/identity/session_cookie_test.go` | 🟡 Planned |
| GR-008 | int | `TestWebhook_when_hmac_invalid_should_400_before_parse` | `/tests/integration/webhook/hmac_test.go` | ✅ Covered (INV-093) |
| GR-009 | int | `TestWebhook_when_event_id_repeated_should_return_200_no_reprocess` | `/tests/integration/webhook/idempotency_test.go` | ✅ Covered (INV-015) |
| GR-010 | unit | `TestI18N_when_key_missing_should_fallback_to_key` | `/tests/unit/i18n/fallback_test.go` | ⚠️ Pendente (A-DC-QA-011) |
| GR-011 | e2e | `GR011_audit_trail_visible_in_admin` | `/tests/e2e/admin/audit_trail.spec.ts` | 🟡 Planned |
| GR-012 | int | `TestPIIFilter_when_email_in_log_should_mask` | `/tests/integration/observability/pii_filter_test.go` | ✅ Covered (INV-092) |
| GR-013 | e2e | `GR013_lighthouse_a11y_score_ge_90_critical_routes` | `/tests/e2e/a11y/lighthouse.spec.ts` | ⚠️ Pendente (A-DC-QA-011) |
| GR-014 | int | `TestUUIDv7_when_generated_should_be_monotonic` | `/tests/unit/pkg/uuid7_test.go` | ✅ Covered (INV-094) |
| GR-015 | int | `TestOutbox_when_publish_should_commit_with_tx` | `/tests/integration/outbox/publish_test.go` | ✅ Covered (INV-095) |
| GR-016 | unit | `TestConfig_when_abac_matrix_empty_should_fail_startup` | `/tests/unit/config/validate_test.go` | ✅ Covered (INV-096) |
| GR-017 | int | `TestOpenAPI_contract_when_handler_drift_should_fail_ci` | `/tests/integration/contract/openapi_test.go` | 🟡 Planned |
| GR-018 | int | `TestSentry_when_panic_should_emit_event` | `/tests/integration/observability/sentry_test.go` | 🟡 Planned |
| GR-019 | int | `TestPresignedURL_when_ttl_lt_24h_should_pass` | `/tests/integration/content/presigned_test.go` | ✅ Covered (INV-070) |
| GR-020 | int | `TestOutbox_drain_when_consumer_fails_should_retry` | `/tests/integration/outbox/retry_test.go` | 🟡 Planned |
| GR-021 | unit | `TestMoney_when_float_assigned_should_not_compile` | `/tests/unit/pkg/money_test.go` | ✅ Covered (INV-014) |
| GR-022 | int | `TestBackup_when_restore_from_snapshot_should_pass_integrity` | `/tests/integration/ops/backup_test.go` | ⏭️ Deferred (M3) |
| GR-023 | int | `TestAuditTrail_when_retention_5y_should_archive` | `/tests/integration/audit/retention_test.go` | ✅ Covered (INV-033, INV-090) |
| GR-024 | e2e | `GR024_rate_limit_auth_20rpm` | `/tests/e2e/security/ratelimit.spec.ts` | 🟡 Planned |
| GR-025 | int | `TestLGPDExport_when_schema_json_published_should_match_v1` | `/tests/integration/lgpd/export_schema_test.go` | ⚠️ Pendente (A-DC-QA-011) |
| GR-026 | int | `TestDPIA_when_executed_should_generate_report` | `/tests/integration/lgpd/dpia_test.go` | ⏭️ Deferred (M3) |
| GR-027 | int | `TestTrial_when_persona_syndic_should_be_15d` | `/tests/integration/billing/trial_persona_test.go` | ✅ Covered (INV-017) |
| GR-028 | int | `TestReputation_when_metrics_change_should_recalc` | `/tests/integration/commercial/reputation_test.go` | ✅ Covered (INV-036) |
| GR-029 | int | `TestVideoLock_when_day89_should_block_replace` | `/tests/integration/content/lock_90d_test.go` | ✅ Covered (INV-056, INV-059) |
| GR-030 | int | `TestMandateClosing_when_dossier_not_ready_should_block_encerramento` | `/tests/integration/institutional/mandate_close_test.go` | 🟡 Planned |
| GR-031 | e2e | `GR031_1device_policy_new_device_invalidates_previous` | `/tests/e2e/identity/device_1.spec.ts` | ✅ Covered (INV-005) |
| GR-032 | unit | `TestMoney_when_int64_cents_only_should_not_accept_float` | `/tests/unit/pkg/money_test.go` | ✅ Covered (INV-014) |
| GR-033 | int | `TestTimezone_when_job_run_should_use_sao_paulo` | `/tests/integration/ops/timezone_test.go` | 🟡 Planned |
| GR-034 | int | `TestFeatureFlag_when_rollout_50_should_split` | `/tests/integration/billing/feature_flag_test.go` | ⏭️ Deferred (M2+) |
| GR-035 | e2e | `GR035_beyond_corp_zero_trust_e2e` | `/tests/e2e/security/zero_trust.spec.ts` | 🟡 Planned |

---

## 3. Identity (IDN-001..IDN-035)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| IDN-001 | int | `TestZitadelCallback_when_first_login_should_upsert_user` | `/tests/integration/identity/callback_test.go` | 🟡 Planned |
| IDN-002 | int | `TestZitadelCallback_when_no_role_should_403_no_upsert` | `/tests/integration/identity/no_role_test.go` | ✅ Covered (INV-004, INV-010) |
| IDN-003 | unit | `TestEmail_when_not_rfc5322_should_reject` | `/tests/unit/identity/email_vo_test.go` | 🟡 Planned |
| IDN-004 | unit | `TestCPF_when_invalid_verifier_should_reject` | `/tests/unit/identity/cpf_vo_test.go` | 🟡 Planned |
| IDN-005 | unit | `TestCNPJ_when_invalid_verifier_should_reject` | `/tests/unit/identity/cnpj_vo_test.go` | 🟡 Planned |
| IDN-006 | int | `TestUser_when_duplicate_cpf_should_conflict` | `/tests/integration/identity/cpf_unique_test.go` | ✅ Covered (INV-002) |
| IDN-007 | int | `TestUser_when_duplicate_cnpj_should_conflict` | `/tests/integration/identity/cnpj_unique_test.go` | ✅ Covered (INV-003) |
| IDN-008 | unit | `TestDeviceFingerprint_when_client_sends_raw_should_hash_server` | `/tests/unit/identity/fingerprint_test.go` | ✅ Covered (INV-009) |
| IDN-009 | int | `TestSession_when_login_second_device_should_revoke_first` | `/tests/integration/identity/session_1device_test.go` | ✅ Covered (INV-005) |
| IDN-010 | int | `TestSession_when_device_change_should_audit_log` | `/tests/integration/identity/device_audit_test.go` | 🟡 Planned |
| IDN-011 | int | `TestPKCE_when_state_mismatch_should_fail` | `/tests/integration/auth/pkce_test.go` | 🟡 Planned |
| IDN-012 | int | `TestMFA_when_admin_login_should_require_stepup` | `/tests/integration/identity/mfa_admin_test.go` | 🟡 Planned |
| IDN-013 | int | `TestMultiContext_when_user_has_memberships_should_list` | `/tests/integration/identity/multi_context_test.go` | 🟡 Planned |
| IDN-014 | int | `TestImpersonate_when_admin_acts_should_audit` | `/tests/integration/identity/impersonate_test.go` | 🟡 Planned |
| IDN-015 | int | `TestRoleBroadcast_when_non_admin_should_403` | `/tests/integration/identity/broadcast_role_test.go` | 🟡 Planned |
| IDN-016 | int | `TestLogout_when_called_should_invalidate_local_and_zitadel` | `/tests/integration/identity/logout_test.go` | 🟡 Planned |
| IDN-017 | int | `TestOnboarding_when_finish_should_create_subscription_trial` | `/tests/integration/identity/onboarding_test.go` | ✅ Covered (cross BIL-003) |
| IDN-018 | unit | `TestRoleType_when_subcontext_should_be_local_only` | `/tests/unit/identity/role_type_test.go` | 🟡 Planned |
| IDN-019 | int | `TestPasswordReset_when_zitadel_flow_should_redirect` | `/tests/integration/identity/password_reset_test.go` | 🟡 Planned |
| IDN-020 | int | `TestEmailVerify_when_not_verified_should_block_publish` | `/tests/integration/identity/email_verified_test.go` | 🟡 Planned |
| IDN-021 | int | `TestSyndicDataFlags_when_registered_should_mirror_matrix` | `/tests/integration/identity/syndic_flags_test.go` | 🟡 Planned |
| IDN-022 | unit | `TestLGPDPseudonym_when_hashed_should_use_keyed_hmac` | `/tests/unit/lgpd/pseudonym_test.go` | ⚠️ Pendente (A-DC-PEN-038) |
| IDN-023 | int | `TestUserDelete_when_soft_should_schedule_hardDelete_15d` | `/tests/integration/lgpd/soft_delete_test.go` | 🟡 Planned |
| IDN-024 | int | `TestBan_when_temporary_should_expire_at_time` | `/tests/integration/identity/ban_test.go` | 🟡 Planned |
| IDN-025 | int | `TestMarketingAgency_when_scoped_token_should_only_access_videos` | `/tests/integration/identity/agency_scope_test.go` | ✅ Covered (INV-049, INV-066) |
| IDN-026 | int | `TestIPAntifraud_when_same_IP_2_accounts_should_flag` | `/tests/integration/identity/ip_antifraud_test.go` | 🟡 Planned |
| IDN-027 | int | `TestInviteAccept_when_token_valid_should_create_membership` | `/tests/integration/identity/invite_test.go` | 🟡 Planned |
| IDN-028 | int | `TestInviteRevoke_when_syndic_revokes_should_invalidate_token` | `/tests/integration/identity/invite_revoke_test.go` | 🟡 Planned |
| IDN-029 | int | `TestAgencyRevocation_when_company_revokes_should_kill_token` | `/tests/integration/identity/agency_revoke_test.go` | 🟡 Planned |
| IDN-030 | int | `TestSessionAudit_when_revoked_should_log_reason` | `/tests/integration/identity/session_audit_test.go` | 🟡 Planned |
| IDN-031 | int | `TestZitadelSync_when_cron_runs_should_reconcile_drift` | `/tests/integration/identity/zitadel_sync_test.go` | 🟡 Planned |
| IDN-032 | int | `TestSessionRegenerate_when_auth_state_changes_should_rotate_id` | `/tests/integration/identity/session_regenerate_test.go` | ⚠️ Pendente (A-DC-PEN-003) |
| IDN-033 | int | `TestPatchMeRole_when_body_contains_role_should_ignore_DTO_strict` | `/tests/integration/identity/mass_assign_test.go` | ⚠️ Pendente (A-DC-PEN-010) |
| IDN-034 | int | `TestConflictOfInterest_when_onboarding_should_require_declaration` | `/tests/integration/identity/conflict_declaration_test.go` | ⚠️ Pendente (A-DC-QA-010 — AC frequência renovação) |
| IDN-035 | int | `TestRoleAssignment_when_zitadel_role_added_should_sync_local` | `/tests/integration/identity/role_sync_test.go` | 🟡 Planned |

---

## 4. Billing (BIL-001..BIL-034)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| BIL-001 | int | `TestPlan_when_persona_filter_should_return_catalog` | `/tests/integration/billing/plan_catalog_test.go` | 🟡 Planned |
| BIL-002 | int | `TestSubscription_when_update_should_append_new_row` | `/tests/integration/billing/subscription_append_test.go` | ✅ Covered (INV-012) |
| BIL-003 | int | `TestTrial_when_onboarding_should_persona_aware_duration` | `/tests/integration/billing/trial_start_test.go` | ✅ Covered (INV-017) |
| BIL-004 | int | `TestTrialBlock_when_strategic_action_should_402` | `/tests/integration/billing/trial_block_test.go` | 🟡 Planned |
| BIL-005 | e2e | `BIL005_trial_banner_T24h` | `/tests/e2e/billing/trial_banner.spec.ts` | 🟡 Planned |
| BIL-006 | int | `TestTrialExpire_when_endDate_passed_should_softblock` | `/tests/integration/billing/trial_expire_test.go` | ✅ Covered (INV-019) |
| BIL-007 | int | `TestCheckout_when_user_clicks_should_create_session` | `/tests/integration/billing/checkout_test.go` | 🟡 Planned |
| BIL-008 | int | `TestCustomerPortal_when_post_should_redirect` | `/tests/integration/billing/portal_test.go` | 🟡 Planned |
| BIL-009 | int | `TestStripeWebhook_when_hmac_invalid_should_400` | `/tests/integration/billing/webhook_hmac_test.go` | ✅ Covered (INV-018, INV-093) |
| BIL-010 | int | `TestStripeWebhook_when_event_id_duplicate_should_return_200_no_reprocess` | `/tests/integration/billing/webhook_idempotency_test.go` | ✅ Covered (INV-015) |
| BIL-011 | int | `TestStripeWebhook_when_event_type_whitelisted_should_route` | `/tests/integration/billing/webhook_routing_test.go` | ⚠️ Pendente (BIL-011 lista exhaustiva) |
| BIL-012 | int | `TestABACCache_when_webhook_affects_plan_should_invalidate` | `/tests/integration/billing/cache_invalidate_test.go` | 🟡 Planned |
| BIL-013 | int | `TestUpgrade_when_plan_changes_should_prorate_via_stripe` | `/tests/integration/billing/prorata_test.go` | 🟡 Planned |
| BIL-014 | int | `TestCancel_when_user_cancels_should_softblock_at_period_end` | `/tests/integration/billing/cancel_test.go` | 🟡 Planned |
| BIL-015 | int | `TestGracePeriod_when_expires_should_block_immediately` | `/tests/integration/billing/no_grace_test.go` | ✅ Covered (INV-019) |
| BIL-016 | pbt | `TestQuotaConnectMe_when_exhausted_should_403` + property | `/tests/property/billing/quota_connect_me_property_test.go` | ✅ Covered (INV-016 — 300 casos) |
| BIL-017 | int | `TestQuotaVideoMonthly_when_N2_publishes_9th_should_403` | `/tests/integration/billing/quota_video_test.go` | ⚠️ Pendente (A-DC-QA-071 reconciliado — ver nota) |
| BIL-018 | int | `TestCondominiumLimit_when_16th_should_403` | `/tests/integration/institutional/condo_limit_test.go` | ✅ Covered (INV-027) |
| BIL-019 | int | `TestVideoCurriculum_when_day89_should_block_replace` | `/tests/integration/content/curriculum_lock_test.go` | ✅ Covered (INV-056) |
| BIL-020 | int | `TestCouponCooldown_when_second_in_4h_should_429` | `/tests/integration/commercial/cooldown_test.go` | ✅ Covered (INV-051) |
| BIL-021 | int | `TestFeatureFlags_when_plan_lookup_should_return_matrix` | `/tests/integration/billing/feature_flags_test.go` | 🟡 Planned |
| BIL-022 | int | `TestRollout_when_50pct_should_split` | `/tests/integration/billing/rollout_test.go` | ⏭️ Deferred (M3) |
| BIL-023 | int | `TestGetMeQuotas_when_user_calls_should_return_used_limit` | `/tests/integration/billing/quotas_endpoint_test.go` | 🟡 Planned |
| BIL-024 | int | `TestQuotaNightlyJob_when_runs_should_materialize_monthly` | `/tests/integration/billing/quota_job_test.go` | ⏭️ Deferred (M3) |
| BIL-025 | int | `TestSmartRetries_when_payment_failed_should_retry_3x_7d` | `/tests/integration/billing/smart_retry_test.go` | ⚠️ Pendente (BIL-025 config exata) |
| BIL-026 | int | `TestInvoicesList_when_cursor_should_page` | `/tests/integration/billing/invoices_test.go` | 🟡 Planned |
| BIL-027 | int | `TestRefund_when_admin_issues_should_audit` | `/tests/integration/billing/refund_test.go` | 🟡 Planned |
| BIL-028 | int | `TestTrial_when_second_subscription_should_not_grant_trial` | `/tests/integration/billing/trial_no_reset_test.go` | ✅ Covered (INV-013) |
| BIL-029 | int | `TestWebhookRetry_when_backend_fails_should_stripe_retry_72h` | `/tests/integration/billing/webhook_retry_test.go` | 🟡 Planned |
| BIL-030 | unit | `TestPlanCurrency_when_not_BRL_should_reject` | `/tests/unit/billing/currency_test.go` | 🟡 Planned |
| BIL-031 | int | `TestStripeTax_backlog` | `/tests/integration/billing/stripe_tax_test.go` | ⏭️ Deferred (backlog W) |
| BIL-032 | int | `TestUmbrellaPlan_backlog` | — | ⏭️ Deferred (backlog W) |
| BIL-033 | int | `TestMercadoPago_backlog` | — | ⏭️ Deferred (backlog W) |
| BIL-034 | int | `TestAdminMRR_when_loaded_should_show_aggregate` | `/tests/integration/billing/admin_mrr_test.go` | 🟡 Planned |

**Nota BIL-017**: após A-DC-QA-071 reconciliado, a quota é **por plano com direito a publicar**: N2=4/mês, N3=12/mês, Plus=8/mês, Pro=12/mês. N1 não publica (matriz §3 ✗). Teste `T-BIL-017-int-quota-video` cobre cenários positivos (N2 no 4º ok, 5º 403) e negativos (N1 tenta publicar → 402/403 `PLAN_REQUIRED` antes da quota).

---

## 5. Institutional (INS-001..INS-043)

Formato abreviado (default heurística quando não explícito):

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| INS-001..INS-005 | int | `TestCondominium_*` (cadastro etapas, endereço, unidades) | `/tests/integration/institutional/condominium_test.go` | 🟡 Planned |
| INS-006 | pbt + int | `TestFracaoIdeal_property_sum_le_100` | `/tests/property/institutional/fracao_property_test.go` | ✅ Covered (INV-021 — 100 casos) |
| INS-007 | unit | `TestFracaoIdeal_VO_range_validation` | `/tests/unit/institutional/fracao_vo_test.go` | ✅ Covered (INV-022) |
| INS-008 | unit | `TestFracaoIdeal_schema_numeric_10_4` | `/tests/unit/institutional/fracao_schema_test.go` | ✅ Covered (INV-023) |
| INS-009 | int | `TestUnit_titular_dependents_max_2` | `/tests/integration/institutional/unit_test.go` | ✅ Covered (INV-029) |
| INS-010 | int | `TestMembership_unique_active_per_user_condo` | `/tests/integration/institutional/membership_test.go` | ✅ Covered (INV-026) |
| INS-011 | int | `TestSyndicMandate_condo_limit_15` | `/tests/integration/institutional/mandate_test.go` | ✅ Covered (INV-027) |
| INS-012 | unit | `TestMandate_end_date_after_start_date` | `/tests/unit/institutional/mandate_vo_test.go` | ✅ Covered (INV-028) |
| INS-013 | int | `TestCondominium_publicID_unique_8chars` | `/tests/integration/institutional/public_id_test.go` | ✅ Covered (INV-030) |
| INS-014 | int | `TestTimelineEntry_immutable_no_update_delete` | `/tests/integration/institutional/timeline_immutable_test.go` | ✅ Covered (INV-024) |
| INS-015 | int | `TestTimelineEntry_correct_creates_new_with_link` | `/tests/integration/institutional/timeline_correct_test.go` | ✅ Covered (INV-032) |
| INS-016 | int | `TestAnnouncement_immutable_after_published` | `/tests/integration/institutional/announcement_test.go` | ✅ Covered (INV-025) |
| INS-017 | int | `TestEvent_create_with_tipos` | `/tests/integration/institutional/event_test.go` | 🟡 Planned |
| INS-018 | int | `TestEvent_ciente_confirmo_registra_notifica_sindico` | `/tests/integration/institutional/event_rsvp_test.go` | 🟡 Planned |
| INS-019 | int | `TestPoll_internal_secret_configurable` | `/tests/integration/institutional/poll_test.go` | 🟡 Planned |
| INS-020 | int | `TestMasterPlan_CRUD_only_via_event` | `/tests/integration/institutional/master_plan_test.go` | ✅ Covered (INV-031) |
| INS-021 | int | `TestMasterPlanActivity_status_via_execution_validated` | `/tests/integration/institutional/master_plan_activity_test.go` | ✅ Covered (INV-031) |
| INS-022 | int | `TestExportPDF_TimelineGestao_when_n2_should_pass` | `/tests/integration/institutional/export_pdf_test.go` | 🟡 Planned |
| INS-023 | int | `TestDossier_when_close_mandate_should_generate` | `/tests/integration/institutional/dossier_test.go` | 🟡 Planned |
| INS-024 | int | `TestCommunicationOfficial_create_N2_plus` | `/tests/integration/institutional/communication_test.go` | 🟡 Planned |
| INS-025 | int | `TestComplianceDeclaration_one_per_year` | `/tests/integration/institutional/compliance_declaration_test.go` | ✅ Covered (INV-034) |
| INS-026 | int | `TestConflictDeclaration_required_onboarding` | `/tests/integration/institutional/conflict_test.go` | ✅ Covered (INV-035) |
| INS-027 | int | `TestAuditTrail_append_only_5y_retention` | `/tests/integration/institutional/audit_retention_test.go` | ✅ Covered (INV-033, INV-090) |
| INS-028 | int | `TestSealSindico_when_approves_partner_should_register` | `/tests/integration/institutional/seal_test.go` | 🟡 Planned |
| INS-029 | int | `TestDependentCRUD_by_sindic` | `/tests/integration/institutional/dependent_test.go` | 🟡 Planned |
| INS-030 | int | `TestCloseMandate_when_reason_should_allow_transition` | `/tests/integration/institutional/close_mandate_test.go` | 🟡 Planned |
| INS-031 | int | `TestHandOver_when_new_sindic_should_inherit_history` | `/tests/integration/institutional/handover_test.go` | 🟡 Planned |
| INS-032 | int | `TestScoreGovernance_internal_calc` | `/tests/integration/institutional/score_test.go` | 🟡 Planned |
| INS-033 | int | `TestScoreCompliance_internal_calc` | `/tests/integration/institutional/score_test.go` | 🟡 Planned |
| INS-034 | int | `TestRiskMap_by_mandate_category_level` | `/tests/integration/institutional/risk_map_test.go` | 🟡 Planned |
| INS-035 | int | `TestCommunicationValidation_sindic_approve_reject_adjust` | `/tests/integration/institutional/comm_validation_test.go` | 🟡 Planned |
| INS-036 | int | `TestValidacaoPendente_queue_aggregate` | `/tests/integration/institutional/pending_queue_test.go` | 🟡 Planned |
| INS-037 | int | `TestSubGuests_max_3_perfiles` | `/tests/integration/institutional/subguests_test.go` | 🟡 Planned |
| INS-038 | int | `TestPermissionsMatrix_17_toggles_persist` | `/tests/integration/institutional/permissions_test.go` | 🟡 Planned |
| INS-039 | int | `TestInviteFlow_when_guest_accepts_should_membership_active` | `/tests/integration/institutional/invite_flow_test.go` | 🟡 Planned |
| INS-040 | e2e | `INS040_onboarding_sindic_novo_condo_S1_S7` | `/tests/e2e/institutional/onboarding_sindic.spec.ts` | 🟡 Planned |
| INS-041 | int | `TestContinuidade_when_ex_sindic_access_should_be_read_only` | `/tests/integration/institutional/handover_read_only_test.go` | 🟡 Planned |
| INS-042 | int | `TestIndicadores_adesao_taxa_calc` | `/tests/integration/institutional/indicadores_test.go` | 🟡 Planned |
| INS-043 | e2e | `INS043_status_unidade_alugada_remove_titular` | `/tests/e2e/morador/status_unidade.spec.ts` | 🟡 Planned |

---

## 6. Commercial (COM-001..COM-045)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| COM-001..004 | int | `TestCompany_CRUD_CNPJ_verify` | `/tests/integration/commercial/company_test.go` | 🟡 Planned |
| COM-005 | int | `TestCNPJ_provider_pleno_M2` | `/tests/integration/commercial/cnpj_provider_test.go` | ⚠️ Pendente (COM-005 provider) |
| COM-006 | int | `TestCompanyProfile_compliance_markers` | `/tests/integration/commercial/profile_test.go` | 🟡 Planned |
| COM-007 | pbt | `TestReputation_determinism_property` | `/tests/property/commercial/reputation_property_test.go` | ✅ Covered (INV-036) |
| COM-008..012 | int | `TestConnectMeFlow_draft_publish_interest_proposal` | `/tests/integration/commercial/connect_me_flow_test.go` | ✅ Covered parcial (INV-045, INV-047, INV-048) |
| COM-013 | pbt + int | `TestVoteSession_TOCTOU_property + unique_per_voter` | `/tests/property/assembly/vote_property_test.go` | ✅ Covered (INV-071 — 50×20 goroutines) |
| COM-014 | int | `TestSupplierVoteSession_unique_per_voter` | `/tests/integration/commercial/supplier_vote_test.go` | ✅ Covered (INV-042) |
| COM-015 | int | `TestProposal_immutable_after_sent` | `/tests/integration/commercial/proposal_immutable_test.go` | ✅ Covered (INV-039) |
| COM-016 | int | `TestProposal_unique_per_request_company` | `/tests/integration/commercial/proposal_unique_test.go` | ✅ Covered (INV-038) |
| COM-017 | int | `TestProposal_validation_sindic_accept_reject_adjust` | `/tests/integration/commercial/proposal_validation_test.go` | ⚠️ Pendente (A-DC-QA-010 — AC ausente) |
| COM-018 | int | `TestClosedDeal_dupla_assinatura_immutable` | `/tests/integration/commercial/closed_deal_test.go` | ✅ Covered (INV-040) |
| COM-019 | int | `TestClosedDeal_auto_publica_timeline` | `/tests/integration/commercial/closed_deal_timeline_test.go` | ✅ Covered (INV-041) |
| COM-020 | int | `TestAmendment_only_way_to_modify_immutable` | `/tests/integration/commercial/amendment_test.go` | ✅ Covered (INV-050) |
| COM-021 | int | `TestExecutionRecord_validate_sindic_integra_timeline` | `/tests/integration/commercial/execution_test.go` | 🟡 Planned |
| COM-022 | pbt + int | `TestCompanyEvaluation_unique_TOCTOU` | `/tests/property/commercial/evaluation_property_test.go` | ✅ Covered (INV-043 — 50×10) |
| COM-023 | int | `TestCompanyEvaluation_anonymous_to_company` | `/tests/integration/commercial/eval_anonymous_test.go` | ✅ Covered (INV-044) |
| COM-024 | int | `TestReputationTier_recalc_on_evaluation` | `/tests/integration/commercial/reputation_recalc_test.go` | ✅ Covered (INV-054) |
| COM-025 | int | `TestReputationTier_promotion_thresholds` | `/tests/integration/commercial/reputation_tier_test.go` | 🟡 Planned (⚠️ A-DC-QA-012 MoSCoW) |
| COM-026 | int | `TestHarassmentReport_confirmed_suspends_company_72h` | `/tests/integration/commercial/harassment_test.go` | ✅ Covered (INV-055) |
| COM-027 | int | `TestPartner_CRUD_categoria_CEP` | `/tests/integration/commercial/partner_test.go` | 🟡 Planned |
| COM-028 | int | `TestCoupon_format_regex` | `/tests/unit/commercial/coupon_code_test.go` | ✅ Covered (INV-052) |
| COM-029 | unit | `TestPalavraDia_max_5_uppercase` | `/tests/unit/commercial/palavra_dia_test.go` | ✅ Covered (INV-053) |
| COM-030 | int | `TestCoupon_cooldown_4h_per_user_partner` | `/tests/integration/commercial/coupon_cooldown_test.go` | ✅ Covered (INV-051) |
| COM-031 | int | `TestCouponGeneration_throttle_429` | `/tests/integration/commercial/coupon_throttle_test.go` | 🟡 Planned |
| COM-032 | int | `TestPartnerProfile_seal_sindico` | `/tests/integration/commercial/seal_test.go` | 🟡 Planned |
| COM-033 | int | `TestPartnerCampaign_sazonal` | `/tests/integration/commercial/campaign_test.go` | 🟡 Planned |
| COM-034 | int | `TestBenefitScope_aberta_bairro_exclusiva_condo` | `/tests/integration/commercial/benefit_scope_test.go` | 🟡 Planned |
| COM-035 | int | `TestMarketingAgency_upload_video_institucional_scoped` | `/tests/integration/commercial/agency_upload_test.go` | ✅ Covered (INV-049, INV-066) |
| COM-036 | int | `TestMarketingAgency_zero_access_to_deals` | `/tests/integration/abac/agency_deny_deals_test.go` | ✅ Covered (INV-049) |
| COM-037 | int | `TestMarketingAgency_cannot_initiate_connect_me` | `/tests/integration/commercial/agency_connectme_deny_test.go` | 🟡 Planned |
| COM-038 | int | `TestConnectMe_auto_close_48h` | `/tests/integration/commercial/auto_close_test.go` | ✅ Covered (INV-047) |
| COM-039 | int | `TestConnectMe_unidirectional_no_reply` | `/tests/integration/commercial/connect_me_unidirectional_test.go` | ✅ Covered (INV-037) |
| COM-040 | int | `TestConnectMe_reveal_lgpd_log_mandatory` | `/tests/integration/commercial/lgpd_log_test.go` | ✅ Covered (INV-048) |
| COM-041 | int | `TestConnectMe_quota_decrement_on_publish` | `/tests/integration/commercial/quota_decrement_test.go` | ✅ Covered (INV-045) |
| COM-042 | int | `TestConnectMe_quota_reset_jan_1` | `/tests/integration/commercial/quota_reset_test.go` | ✅ Covered (INV-046) |
| COM-043 | int | `TestCompanyAssessment_req26_anonymous` | `/tests/integration/commercial/assessment_test.go` | ✅ Covered (INV-044) |
| COM-044 | int | `TestFeatureW_backlog` | — | ⚠️ Pendente (A-DC-QA-010 AC) + ⏭️ Deferred |
| COM-045 | int | `TestResumeVisualization_empresa_plus_pro` | `/tests/integration/commercial/resume_view_test.go` | ⚠️ Pendente (A-DC-QA-070) |

---

## 7. Content (CNT-001..CNT-035)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| CNT-001..005 | int | `TestMuxUpload_saga_compensate_on_failure` | `/tests/integration/content/mux_saga_test.go` | ✅ Covered (INV-065) |
| CNT-006 | int | `TestMuxWebhook_hmac_before_parse` | `/tests/integration/content/mux_webhook_test.go` | ✅ Covered (INV-064) |
| CNT-007 | int | `TestMuxWebhook_idempotency` | `/tests/integration/content/mux_idempotency_test.go` | ✅ Covered (INV-015) |
| CNT-008 | int | `TestVideoModeration_SLA_48h` | `/tests/integration/content/moderation_sla_test.go` | ⚠️ Pendente (P-SM-002) |
| CNT-009 | int | `TestVideoState_transition_machine` | `/tests/integration/content/video_state_test.go` | ✅ Covered (INV-058) |
| CNT-010 | int | `TestVideoPublish_sets_lock_90d` | `/tests/integration/content/lock_90d_test.go` | ✅ Covered (INV-056) |
| CNT-011 | int | `TestVideoReplace_after_90d_allowed_quarterly` | `/tests/integration/content/replace_90d_test.go` | ✅ Covered (INV-059) |
| CNT-012 | int | `TestVideoMetrics_cross_tenant_never_leak` | `/tests/integration/abac/cross_tenant_metrics_test.go` | ✅ Covered (INV-060 — 100+ casos) |
| CNT-013 | int | `TestRemoveVideo_pre_90d_requires_admin_audit` | `/tests/integration/content/admin_bypass_test.go` | ✅ Covered (INV-057) |
| CNT-014 | int | `TestAgencyUpload_scoped_to_company` | `/tests/integration/content/agency_upload_test.go` | ✅ Covered (INV-066) |
| CNT-015 | int | `TestFeature_backlog_CNT015` | — | ⚠️ Pendente (A-DC-QA-010 AC) |
| CNT-016 | int | `TestEnterprisePro_create_certified_course` | `/tests/integration/content/pro_course_test.go` | 🟡 Planned |
| CNT-017 | int | `TestCertificate_auto_gen_on_100pct` | `/tests/integration/content/certificate_test.go` | ✅ Covered (INV-068) |
| CNT-018 | int | `TestEnrollment_progress_tracking` | `/tests/integration/content/enrollment_test.go` | 🟡 Planned |
| CNT-019 | int | `TestEbook_access_by_plan` | `/tests/integration/content/ebook_test.go` | ✅ Covered (INV-069) |
| CNT-020 | int | `TestLMSAccess_by_persona_C_M3` | `/tests/integration/content/lms_access_test.go` | 🟡 Planned (A-DC-QA-013) |
| CNT-021 | int | `TestLivestream_restricted_admin_M1` | `/tests/integration/content/livestream_test.go` | 🟡 Planned |
| CNT-022 | int | `TestBroadcastAdmin_non_admin_403` | `/tests/integration/content/broadcast_test.go` | 🟡 Planned |
| CNT-023 | int | `TestPlaybackURL_TTL_le_24h` | `/tests/integration/content/playback_ttl_test.go` | ✅ Covered (INV-070) |
| CNT-024 | int | `TestVideoCurriculum_90s_unique_per_user` | `/tests/integration/content/curriculum_test.go` | ✅ Covered (INV-056, INV-059) |
| CNT-025 | int | `TestResidentPaid_cursos_certificados_access` | `/tests/integration/content/resident_paid_courses_test.go` | 🟡 Planned |
| CNT-026 | int | `TestBaseResident_preview_25pct_paywall` | `/tests/integration/content/preview_test.go` | 🟡 Planned |
| CNT-027 | int | `TestNeighborhoodBenefit_feed_by_CEP` | `/tests/integration/content/feed_cep_test.go` | 🟡 Planned |
| CNT-028 | int | `TestLive_recorded_replay_30d` | `/tests/integration/content/live_replay_test.go` | 🟡 Planned |
| CNT-029 | int | `TestVisibilityGrant_revoked_on_vote_closed` | `/tests/integration/content/visibility_grant_test.go` | ✅ Covered (INV-063) |
| CNT-030 | int | `TestVideoCurriculum_upload_pagante` | `/tests/integration/content/curriculum_upload_test.go` | 🟡 Planned |
| CNT-031 | int | `TestVideoForum_role_gate_N1_plus` | `/tests/integration/content/forum_test.go` | 🟡 Planned |
| CNT-032 | int | `TestParceiroVizinhanca_no_LMS_access` | `/tests/integration/content/partner_no_lms_test.go` | ✅ Covered (INV-067) |
| CNT-033 | int | `TestBlindSocial_others_cannot_see_counters` | `/tests/integration/content/blind_social_test.go` | 🟡 Planned |
| CNT-034 | int | `TestMarketingLink_publisher` | `/tests/integration/content/marketing_link_test.go` | 🟡 Planned |
| CNT-035 | int | `TestYouTubePodcast_embedded_integration` | `/tests/integration/content/podcast_test.go` | 🟡 Planned |

---

## 8. Assembly (ASM-001..ASM-033)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| ASM-001 | int | `TestAssembly_create_scheduled_type` | `/tests/integration/assembly/create_test.go` | 🟡 Planned |
| ASM-002 | int | `TestAssembly_publish_edital_pauta` | `/tests/integration/assembly/publish_test.go` | 🟡 Planned |
| ASM-003 | int | `TestAssembly_start_requires_edital_published` | `/tests/integration/assembly/start_requires_edital_test.go` | ✅ Covered (INV-074) |
| ASM-004 | int | `TestAssembly_one_active_per_condo` | `/tests/integration/assembly/one_active_test.go` | ✅ Covered (INV-075) |
| ASM-005 | pbt | `TestVote_unique_per_item_voter_TOCTOU` | `/tests/property/assembly/vote_unique_property_test.go` | ✅ Covered (INV-071) |
| ASM-006 | int | `TestAssembly_homologation_required_after_close` | `/tests/integration/assembly/homologation_test.go` | ✅ Covered (INV-085) |
| ASM-007 | int | `TestMinutes_immutable_after_published` | `/tests/integration/assembly/minutes_immutable_test.go` | ✅ Covered (INV-072) |
| ASM-008 | pbt + int | `TestQuorum_by_fracao_ideal` | `/tests/property/assembly/quorum_property_test.go` | ✅ Covered (INV-073) |
| ASM-009 | int | `TestVoteTimeout_registers_absent_idempotent` | `/tests/integration/assembly/vote_timeout_test.go` | ✅ Covered (INV-076) |
| ASM-010 | int | `TestProxy_cannot_vote_different_titular` | `/tests/integration/assembly/proxy_test.go` | ✅ Covered (INV-077) |
| ASM-011 | unit | `TestProxy_valid_24h_after_end` | `/tests/unit/assembly/proxy_vo_test.go` | ✅ Covered (INV-078) |
| ASM-012 | unit | `TestProxy_revoke_1h_before_start` | `/tests/unit/assembly/proxy_revoke_test.go` | ✅ Covered (INV-079) |
| ASM-013 | int | `TestAgenda_published_is_read_only` | `/tests/integration/assembly/agenda_read_only_test.go` | ✅ Covered (INV-080) |
| ASM-014 | int | `TestPresident_cannot_vote` | `/tests/integration/assembly/president_no_vote_test.go` | ✅ Covered (INV-081) |
| ASM-015 | unit | `TestVote_snapshot_fracao_ideal_weight` | `/tests/unit/assembly/vote_snapshot_test.go` | ✅ Covered (INV-083) |
| ASM-016 | int | `TestLivekit_saga_compensate_EndRoom_on_fail` | `/tests/integration/assembly/live_saga_test.go` | ✅ Covered (INV-084) |
| ASM-017 | int | `TestCheckin_QR_terminal_manual_register_attendance` | `/tests/integration/assembly/checkin_test.go` | 🟡 Planned |
| ASM-018 | e2e | `ASM018_assembly_live_e2e_AS1_AS8` | `/tests/e2e/assembly/live_flow.spec.ts` | 🟡 Planned |
| ASM-019 | int | `TestLivekitWebhook_room_events` | `/tests/integration/assembly/livekit_webhook_test.go` | 🟡 Planned |
| ASM-020 | int | `TestRaiseHand_queue_ordering` | `/tests/integration/assembly/raise_hand_test.go` | 🟡 Planned |
| ASM-021 | int | `TestMinutes_preview_auto_generate` | `/tests/integration/assembly/minutes_preview_test.go` | 🟡 Planned |
| ASM-022 | int | `TestDigitalSignature_MP_2200_2` | `/tests/integration/assembly/signature_test.go` | 🟡 Planned |
| ASM-023 | int | `TestCancel_upto_3d_before` | `/tests/integration/assembly/cancel_test.go` | 🟡 Planned |
| ASM-024 | int | `TestAbsent_15min_momentaneo` | `/tests/integration/assembly/absent_15min_test.go` | 🟡 Planned |
| ASM-025 | int | `TestNotification_T5_T3_T1` | `/tests/integration/assembly/notifications_test.go` | 🟡 Planned |
| ASM-026 | int | `TestVoteResult_immutable_after_close` | `/tests/integration/assembly/result_immutable_test.go` | 🟡 Planned |
| ASM-027 | int | `TestVoteAggregation_weighted_by_fracao` | `/tests/integration/assembly/aggregation_test.go` | 🟡 Planned |
| ASM-028 | int | `TestDossierExport_PDF_complete` | `/tests/integration/assembly/dossier_export_test.go` | 🟡 Planned |
| ASM-029 | int | `TestEscalation_pauta_item_types` | `/tests/integration/assembly/pauta_types_test.go` | 🟡 Planned |
| ASM-030 | int | `TestAssemblyTypes_ordinaria_extraordinaria_ratificacao` | `/tests/integration/assembly/types_test.go` | 🟡 Planned |
| ASM-031 | int | `TestModality_presential_M1_only` | `/tests/integration/assembly/modality_test.go` | 🟡 Planned |
| ASM-032 | int | `TestHomologationQuorumQualified` | `/tests/integration/assembly/homologation_quorum_test.go` | ⚠️ Pendente (P-INV-007) |
| ASM-033 | int | `TestAssemblyAudit_trail_visible` | `/tests/integration/assembly/audit_test.go` | 🟡 Planned |

---

## 9. Cross-domain (XD-001..XD-035)

| Req ID | Tipo | Test function name | Arquivo | Status |
|---|---|---|---|---|
| XD-001 | int | `TestDomainEvent_UUIDv7_monotonic` | `/tests/integration/xd/event_uuid_test.go` | ✅ Covered (INV-094) |
| XD-002 | int | `TestOutbox_publish_within_tx` | `/tests/integration/xd/outbox_test.go` | ✅ Covered (INV-095) |
| XD-003 | int | `TestEventDispatcher_subscribe_fanout` | `/tests/integration/xd/dispatcher_test.go` | 🟡 Planned |
| XD-004 | pbt | `TestABAC_engine_order_strict` | `/tests/property/abac/order_property_test.go` | ✅ Covered (INV-086 — 400 casos) |
| XD-005 | int | `TestABAC_default_deny_when_unconfigured` | `/tests/integration/abac/default_deny_test.go` | ✅ Covered (INV-087) |
| XD-006 | int | `TestABAC_cache_invalidation_on_role_change` | `/tests/integration/abac/cache_test.go` | 🟡 Planned (A-DC-QA-050 Zitadel) |
| XD-007 | int | `TestABAC_seed_policies_matrix_all_features` | `/tests/integration/abac/seed_matrix_test.go` | ⚠️ Pendente (A-DC-QA-072) |
| XD-008 | int | `TestAdminBypass_emits_audit` | `/tests/integration/abac/admin_bypass_test.go` | ✅ Covered (INV-088) |
| XD-009 | int | `TestTenantIsolation_100plus_cross_tenant` | `/tests/integration/abac/tenant_isolation_test.go` | ✅ Covered (INV-089) |
| XD-010 | int | `TestAuditTrail_append_only` | `/tests/integration/xd/audit_append_test.go` | ✅ Covered (INV-090) |
| XD-011 | int | `TestLGPD_retention_5y_expire_hard_delete` | `/tests/integration/lgpd/retention_test.go` | ✅ Covered (INV-091) |
| XD-012 | int | `TestPII_never_in_logs_static_check` | `/tests/integration/static/pii_log_test.go` | ✅ Covered (INV-092) |
| XD-013 | int | `TestWebhook_hmac_always_before_parse` | `/tests/integration/webhook/hmac_always_test.go` | ✅ Covered (INV-093) |
| XD-014 | int | `TestRateLimit_auth_20rpm_api_60rpm` | `/tests/integration/middleware/ratelimit_test.go` | ✅ Covered (INV-097) |
| XD-015 | int | `TestCORS_star_blocked_staging_prod` | `/tests/integration/middleware/cors_test.go` | ✅ Covered (INV-098) |
| XD-016 | int | `TestCookie_httpOnly_secure_samesite_lax` | `/tests/integration/middleware/cookie_test.go` | ✅ Covered (INV-100) |
| XD-017 | int | `TestSecrets_gitignored_ci_secret_scanner` | `/tests/integration/static/secrets_scan_test.go` | ✅ Covered (INV-099) |
| XD-018 | int | `TestConfig_validate_startup_fail_on_invalid` | `/tests/integration/config/validate_startup_test.go` | ✅ Covered (INV-096) |
| XD-019 | int | `TestFCM_register_device_optin_optout_LGPD` | `/tests/integration/providers/fcm_optin_test.go` | ⚠️ Pendente (A-DC-QA-052) |
| XD-020 | int | `TestDLQ_when_webhook_fails_3x_should_enqueue_dlq` | `/tests/integration/xd/dlq_test.go` | 🟡 Planned |
| XD-021 | int | `TestXD021_cross_ref_INV_based` | `/tests/integration/xd/cross_ref_test.go` | ⚠️ Pendente (A-DC-QA-010 AC delegado) |
| XD-022 | int | `TestXD022_delegates_to_IDN_026_028` | `/tests/integration/xd/delegation_test.go` | ⚠️ Pendente (AC delegado) |
| XD-023 | int | `TestXD023_delegates_to_CNT_009` | `/tests/integration/xd/delegation_test.go` | ⚠️ Pendente (AC delegado) |
| XD-024..XD-030 | int | `TestXD024_030_cross_references_consolidated` | `/tests/integration/xd/consolidated_test.go` | ⚠️ Pendente (AC delegado — A-DC-QA-010) |
| XD-031 | int | `TestOutboxDrainWorker_NATS_at_least_once` | `/tests/integration/xd/outbox_nats_test.go` | 🟡 Planned (A-DC-QA-051) |
| XD-032 | int | `TestOpenSearch_reindex_video_published_event` | `/tests/integration/xd/opensearch_test.go` | 🟡 Planned (A-DC-QA-051) |
| XD-033 | int | `TestS3_presigned_PUT_GET_lifecycle` | `/tests/integration/xd/s3_test.go` | 🟡 Planned (A-DC-QA-051) |
| XD-034 | int | `TestZitadel_actions_webhook_invalidate_abac` | `/tests/integration/identity/zitadel_actions_test.go` | 🟡 Planned (A-DC-QA-050) |
| XD-035 | int | `TestSaga_orchestration_default_compensation` | `/tests/integration/saga/orchestration_test.go` | 🟡 Planned (ADR-0030) |

---

## 10. Invariantes (INV-001..INV-100) — resumo de cobertura

Toda linha de invariante em `01-domain/invariants.md` já aponta `Teste`. Consolidado aqui pra referência cruzada:

| Range | BC | PBT obrig. | Int obrig. | Unit obrig. | Gaps |
|---|---|---|---|---|---|
| INV-001..010 | identity | — | INV-001..007 | INV-004, INV-008, INV-009 | — |
| INV-011..020 | billing | **INV-016** | INV-011..020 | INV-013, INV-014, INV-017 | — |
| INV-021..035 | institutional | **INV-021** | INV-021..035 | INV-022, INV-029, INV-032 | — |
| INV-036..055 | commercial | **INV-043** | INV-036..055 | INV-036, INV-039, INV-050, INV-052, INV-053 | PBT recomendado (A-DC-QA-021): INV-042, INV-047, INV-051, INV-059, INV-076, INV-077 |
| INV-056..070 | content | — | INV-056..070 | INV-056, INV-058, INV-059, INV-070 | — |
| INV-071..085 | assembly | **INV-071** | INV-071..085 | INV-077, INV-078, INV-079, INV-083 | — |
| INV-086..100 | cross-domain | **INV-086** | INV-086..100 | INV-094, INV-096 | — |

**5 PBTs obrigatórios implementados**: INV-016, INV-021, INV-043, INV-071, INV-086 (ver [[pbt]]).

---

## 11. State Machines — transições (sumário; detalhe em `01-domain/state-machines.md`)

Cada state machine tem tabela "Cobertura de transições" na própria seção em [[../01-domain/state-machines]] (fechamento A-DC-QA-060). Sumário aqui:

| SM | # Transições | Test IDs | Arquivo |
|---|---|---|---|
| Subscription | 10 | `T-BIL-sm-trial-active`, `T-BIL-sm-trial-expired`, `T-BIL-sm-active-past_due`, `T-BIL-sm-past_due-active`, `T-BIL-sm-past_due-expired`, `T-BIL-sm-active-canceled`, `T-BIL-sm-expired-active`, `T-BIL-sm-canceled-active`, `T-BIL-sm-any-paused`, `T-BIL-sm-no-trial-revival` | `/tests/integration/billing/sm_subscription_test.go` |
| VideoModeration | 11 | `T-CNT-sm-upload-processing`, `T-CNT-sm-proc-moderation`, `T-CNT-sm-mod-approved`, `T-CNT-sm-mod-rejected`, `T-CNT-sm-approved-published`, `T-CNT-sm-published-locked`, `T-CNT-sm-locked-unlocked`, `T-CNT-sm-unlocked-replace`, `T-CNT-sm-remove-admin-bypass`, `T-CNT-sm-no-upload-direct-published`, `T-CNT-sm-rejected-no-republish` | `/tests/integration/content/sm_video_test.go` |
| Assembly | 6 | `T-ASM-sm-scheduled-published`, `T-ASM-sm-published-inProgress`, `T-ASM-sm-inProgress-closed`, `T-ASM-sm-closed-homologated`, `T-ASM-sm-cancel-terminal`, `T-ASM-sm-homologated-immutable` | `/tests/integration/assembly/sm_assembly_test.go` |
| ConnectMeRequest | 10 | `T-COM-sm-draft-published`, `T-COM-sm-published-interest`, `T-COM-sm-published-autoclose`, `T-COM-sm-interest-proposals`, `T-COM-sm-proposals-selected`, `T-COM-sm-selected-closedDeal`, `T-COM-sm-closedDeal-evaluated`, `T-COM-sm-cancel-preSign`, `T-COM-sm-published-draft-forbidden`, `T-COM-sm-closedDeal-cancel-forbidden` | `/tests/integration/commercial/sm_connect_me_test.go` |
| VoteSession | 4 | `T-ASM-sm-vote-open`, `T-ASM-sm-vote-count`, `T-ASM-sm-vote-closed`, `T-ASM-sm-vote-tallied` | `/tests/integration/assembly/sm_vote_session_test.go` |
| Company Reputation | 5 | `T-COM-sm-rep-bronze-prata`, `T-COM-sm-rep-prata-ouro`, `T-COM-sm-rep-ouro-platina`, `T-COM-sm-rep-platina-diamante`, `T-COM-sm-rep-degradation-reversible` | `/tests/integration/commercial/sm_reputation_test.go` |
| User Session | 7 | `T-IDN-sm-login-active`, `T-IDN-sm-active-idle`, `T-IDN-sm-idle-active`, `T-IDN-sm-1device-revoke`, `T-IDN-sm-logout`, `T-IDN-sm-admin-revoke`, `T-IDN-sm-expired-terminal` | `/tests/integration/identity/sm_session_test.go` |
| MasterPlan Activity | 6 | `T-INS-sm-mpa-planned-contracted`, `T-INS-sm-mpa-contracted-inProgress`, `T-INS-sm-mpa-inProgress-completed`, `T-INS-sm-mpa-inProgress-delayed`, `T-INS-sm-mpa-inProgress-rejected`, `T-INS-sm-mpa-amended` | `/tests/integration/institutional/sm_master_plan_test.go` |
| Proposal | 8 | `T-COM-sm-prop-draft-sent`, `T-COM-sm-prop-sent-awaiting`, `T-COM-sm-prop-awaiting-validated`, `T-COM-sm-prop-awaiting-rejected`, `T-COM-sm-prop-awaiting-expired`, `T-COM-sm-prop-validated-accepted`, `T-COM-sm-prop-accepted-immutable`, `T-COM-sm-prop-rejected-terminal` | `/tests/integration/commercial/sm_proposal_test.go` |
| HarassmentReport | 6 | `T-COM-sm-hr-reported-review`, `T-COM-sm-hr-review-confirmed`, `T-COM-sm-hr-review-dismissed`, `T-COM-sm-hr-review-escalated`, `T-COM-sm-hr-confirmed-closed`, `T-COM-sm-hr-closed-terminal` | `/tests/integration/commercial/sm_harassment_test.go` |

**Total**: ~73 transições × 1 test_id cada = 73 testes de state machine. Detalhe canônico na seção "Cobertura de transições" de cada SM em [[../01-domain/state-machines]].

---

## 12. Features da matriz funcional × tests ABAC

Cada linha da matriz (~80 features × 10 personas = 800 células) deve ter **pelo menos 1 caso positivo + 1 negativo** no teste ABAC (A-DC-QA-072). Hoje coberto genericamente via INV-086 (400 casos PBT). Plano de expansão M2:

- Criar `/tests/integration/abac/matrix_feature_by_feature_test.go` com data table completa da matriz.
- Test-id pattern: `T-ABAC-matrix-<modulo>-<feature-slug>-<persona>`.
- Exemplo: `T-ABAC-matrix-pub-video-institucional-N2-allowed` / `...-N1-denied`.

---

## 13. Resumo de cobertura

| Categoria | Total | ✅ Covered | 🟡 Planned | ⚠️ Pendente | ⏭️ Deferred |
|---|---|---|---|---|---|
| Global (GR) | 35 | 13 | 17 | 3 | 2 |
| Identity (IDN) | 35 | 7 | 22 | 4 | 2 |
| Billing (BIL) | 34 | 10 | 15 | 4 | 5 |
| Institutional (INS) | 43 | 18 | 24 | 1 | 0 |
| Commercial (COM) | 45 | 22 | 17 | 5 | 1 |
| Content (CNT) | 35 | 13 | 20 | 2 | 0 |
| Assembly (ASM) | 33 | 17 | 15 | 1 | 0 |
| Cross-domain (XD) | 35 | 16 | 8 | 10 | 1 |
| **Total** | **295** | **116** | **138** | **30** | **11** |

**Cobertura declarada**: 116/295 ≈ **39%** já `✅ Covered` (apontam test_id em `pbt.md`/`integration.md`/`unit.md`/`e2e.md`); 138 `🟡 Planned` com arquivo previsto; 30 `⚠️ Pendente` dependem de reconciliação (A-DC-QA-010/011/012/071/A-DC-PEN-003/010/038/etc); 11 `⏭️ Deferred` pós-M1.

> O objetivo **M1**: sair com ≥ 80% `✅ Covered` pros BCs `identity`, `billing`, `institutional`, `assembly` (BCs no caminho crítico do cliente pioneiro Master Síndico). Remanescentes migram pra M2.

---

## 14. Como atualizar esta matriz

1. Novo requirement (GR-### / BC-###) criado → **obrigatório** adicionar linha aqui com test_id sugerido (heurística §1.3 se domínio desconhecido).
2. Teste efetivamente implementado → promover status de `🟡 Planned` → `✅ Covered`.
3. Pendência resolvida (A-DC-###) → atualizar status e remover referência à pendência.
4. Requirement `W` / deferred → marcar `⏭️ Deferred` com marco.
5. CI pipeline: script `scripts/test-traceability-check.py` compara linhas desta matriz com arquivos reais em `/tests/**` (gate M1 — qualquer `✅ Covered` sem arquivo real = fail).

---

## 15. Links

- [[_moc]]
- [[test-strategy]]
- [[unit]]
- [[integration]]
- [[e2e]]
- [[pbt]]
- [[coverage-thresholds]]
- [[../01-domain/invariants]]
- [[../01-domain/state-machines]]
- [[../01-domain/aggregates/_moc]]
- [[../04-requirements/global]]
- [[../04-requirements/functional/_moc]]
- [[../04-requirements/matrix-functional]]
- [[../11-audit/AUDIT]]
- [[../11-audit/audit-log/2026-04-23-qa-review]] (seed do finding A-DC-QA-001)
