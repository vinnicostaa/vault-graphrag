---
title: E2E tests — Master Síndico
type: guide
tags: [testing, e2e, playwright, patrol, flutter, smoke, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 90-ingested
created: 2026-04-23
updated: 2026-04-23
aliases: [e2e-tests-master-sindico]
---

# E2E tests — Master Síndico v2

End-to-end tests pro **web (SolidJS + Playwright)** e **mobile (Flutter + Patrol / `integration_test`)**. Cobrem fluxos críticos de persona (signup síndico, Connect Me, upload vídeo Mux, assembleia live). **Poucos, caros, protegidos** — 10% da pirâmide.

> Herda [[test-strategy]] §2. Cenários derivam de [[../00-product/portfolio-de-produtos]] (jornadas por sub-produto).

---

## 1. Filosofia E2E

- **Custo alto** (5-60s por teste) → apenas jornadas críticas contratuais.
- **Smoke tests pós-deploy** — validam infra + auth + DB up.
- **Regressão semanal** — full suite roda antes de release.
- **Dados isolados**: cada run usa tenant/user descartável (factory via API).
- **Sem mocks**: integra tudo — backend real, Stripe/Mux/LiveKit em sandbox/test mode.

---

## 2. Web (Playwright)

### Setup

```
web/tests/e2e/
├── playwright.config.ts
├── fixtures/
│   ├── api.fixture.ts       ← cria tenant/usuário via API (setup)
│   └── auth.fixture.ts      ← login Zitadel + cookie session
├── specs/
│   ├── signup-sindico.spec.ts
│   ├── connect-me.spec.ts
│   ├── upload-video.spec.ts
│   ├── assembly-live.spec.ts
│   └── smoke.spec.ts        ← roda pós-deploy
└── teardown/
    └── cleanup.ts           ← deleta tenant/user gerados
```

### Config `playwright.config.ts`

```ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './specs',
  timeout: 60_000,
  retries: 2,               // E2E flaky-tolerant; unit/integration = 0
  workers: 3,
  reporter: [['html'], ['json', { outputFile: 'results.json' }]],
  use: {
    baseURL: process.env.E2E_BASE_URL ?? 'http://localhost:3000',
    screenshot: 'only-on-failure',
    trace: 'retain-on-failure',
    video: 'retain-on-failure',
  },
  projects: [
    { name: 'chromium', use: devices['Desktop Chrome'] },
    { name: 'mobile-chrome', use: devices['Pixel 7'] },
  ],
});
```

### Cenários críticos (M1-M3)

#### Signup síndico → onboard condomínio → dashboard

```ts
test('sindico completa onboarding 6 telas e chega no dashboard', async ({ page, api }) => {
  await page.goto('/auth/signup');
  await page.fill('[name=email]', `sindico+${Date.now()}@ms-test.com`);
  await page.fill('[name=password]', 'Passw0rd!');
  await page.click('button:has-text("Continuar")');

  // Zitadel PKCE callback
  await page.waitForURL(/\/onboarding\/tipo/);

  await page.click('[data-persona=sindico]');
  await page.fill('[name=cpf]', '123.456.789-09');
  await page.click('button:has-text("Próximo")');

  // Criar condomínio
  await page.fill('[name=condominio_nome]', 'Residencial Teste');
  await page.fill('[name=condominio_cnpj]', '11.222.333/0001-81');
  // ... passos restantes

  await expect(page).toHaveURL(/\/dashboard/);
  await expect(page.locator('text=Bem-vindo')).toBeVisible();
});
```

#### Connect Me end-to-end (criar RFP → empresa expressa interesse → LGPD log)

```ts
test('Connect Me flow sindico->empresa dispara LGPD log ao revelar dados', async ({ page, api }) => {
  const { sindico, empresa, condo } = await api.setupConnectMeFixture();

  // Login síndico
  await api.loginAs(page, sindico);
  await page.goto(`/condominios/${condo.id}/connect-me/novo`);
  await page.selectOption('[name=categoria]', 'hidraulica');
  await page.fill('[name=descricao]', 'Vazamento no 3º andar');
  await page.click('button:has-text("Publicar")');

  const rfpId = await page.getAttribute('[data-rfp-id]', 'data-rfp-id');

  // Empresa: expressar interesse
  await api.loginAs(page, empresa);
  await page.goto(`/marketplace/connect-me/${rfpId}`);
  await page.click('button:has-text("Tenho Interesse")');
  await expect(page.locator('text=Dados do síndico revelados')).toBeVisible();

  // Valida LGPD log via API (INV-048, BR-022)
  const logs = await api.getLGPDLogs({ rfpId, action: 'data_revealed' });
  expect(logs).toHaveLength(1);
});
```

#### Upload vídeo Mux (BR-034)

```ts
test('vídeo empresa upload → moderation → approved → published → locked 90d', async ({ page, api }) => {
  const { empresa } = await api.setupEmpresa();
  await api.loginAs(page, empresa);

  await page.goto('/empresa/videos/novo');
  await page.setInputFiles('[name=file]', 'fixtures/video-sample.mp4');
  await page.click('button:has-text("Enviar")');
  await expect(page.locator('text=Em moderação')).toBeVisible({ timeout: 60_000 });

  // Admin aprova
  const admin = await api.createAdmin();
  await api.loginAs(page, admin);
  await page.goto('/admin/moderacao');
  await page.click(`[data-video-id="${videoId}"] button:has-text("Aprovar")`);

  // Empresa publica
  await api.loginAs(page, empresa);
  await page.goto('/empresa/videos');
  await page.click(`[data-video-id="${videoId}"] button:has-text("Publicar")`);

  // Tentar remover antes de 90d (INV-057) → erro
  await page.click(`[data-video-id="${videoId}"] button:has-text("Remover")`);
  await expect(page.locator('text=Trava de 90 dias')).toBeVisible();
});
```

#### Assembleia live (quórum por fração, vote único)

```ts
test('assembleia publica edital → quorum fracional atingido → voto → ata homologada', async ({ browser, api }) => {
  const { sindico, moradores, condo } = await api.setupAssemblyFixture({ units: 10 });
  const assembly = await api.createAssembly({ condoId: condo.id, agenda: [{ title: 'Reforma fachada' }] });

  // Síndico publica edital
  const sindicoCtx = await browser.newContext();
  const sPage = await sindicoCtx.newPage();
  await api.loginAs(sPage, sindico);
  await sPage.goto(`/assembleias/${assembly.id}`);
  await sPage.click('button:has-text("Publicar Edital")');
  await sPage.click('button:has-text("Iniciar")');

  // 7 moradores check-in + voto → Σ fração > 50%
  for (const morador of moradores.slice(0, 7)) {
    const ctx = await browser.newContext();
    const p = await ctx.newPage();
    await api.loginAs(p, morador);
    await p.goto(`/assembleias/${assembly.id}/ao-vivo`);
    await p.click('button:has-text("Sim")'); // voto
  }

  // Síndico encerra + homologa
  await sPage.reload();
  await sPage.click('button:has-text("Encerrar")');
  await sPage.click('button:has-text("Homologar Ata")');
  await expect(sPage.locator('text=Ata homologada')).toBeVisible();
});
```

---

## 3. Mobile (Flutter `integration_test` / Patrol)

### Setup

```
app/integration_test/
├── login_flow_test.dart
├── push_notification_test.dart
├── offline_sync_test.dart
└── assembly_live_test.dart
```

### Exemplo — login + refresh token

```dart
// integration_test/login_flow_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('login Zitadel PKCE → recebe 401 → refresh token transparente', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    await tester.enterText(find.byKey(const Key('email')), 'morador@test.com');
    await tester.enterText(find.byKey(const Key('password')), 'Passw0rd!');
    await tester.tap(find.byKey(const Key('login_button')));
    await tester.pumpAndSettle();

    expect(find.text('Home'), findsOneWidget);

    // força token expiry via helper debug
    await TestHelpers.expireToken();

    await tester.tap(find.byKey(const Key('load_profile')));
    await tester.pumpAndSettle();

    // refresh deve ter acontecido sem pedir login de novo
    expect(find.text('Meu Perfil'), findsOneWidget);
  });
}
```

### Patrol (para fluxos que tocam OS-level — push, biometria)

```dart
patrolTest('push notification abre tela correta', ($) async {
  await $.pumpWidgetAndSettle(app.MyApp());
  await $.native.tapOnNotificationBySelector(NativeSelector(text: 'Nova assembleia'));
  expect($(AssemblyLiveScreen), findsOneWidget);
});
```

---

## 4. Smoke tests (pós-deploy)

Spec isolado `smoke.spec.ts` — roda após CD em staging/prod, < 30s total.

```ts
test('smoke: health, auth, DB, Redis, Stripe', async ({ request }) => {
  expect((await request.get('/healthz')).status()).toBe(200);
  expect((await request.get('/readyz')).status()).toBe(200);

  const token = await getE2ETestToken(); // fixture pré-criada
  const me = await request.get('/api/v1/me', { headers: { Authorization: `Bearer ${token}` } });
  expect(me.status()).toBe(200);
});
```

Acionado via GitHub Actions após `deploy.yml` concluir.

---

## 5. Data management

- **Tenant isolado** por run: `tenant:e2e-<timestamp>-<runId>`.
- **Cleanup obrigatório** no `afterAll`: API `DELETE /api/v1/admin/tenants/:id`.
- **Sandbox externo**: Stripe `sk_test_*`; Mux environment `test`; LiveKit project dev.
- **Nenhum CPF/CNPJ real** — geradores algorítmicos.
- **Screenshots/videos retidos** apenas em falha (LGPD).

---

## 6. Flakiness

E2E aceita `retries: 2` (diferente de unit/integration = 0). Mas:

- Falha sistemática em 3 runs consecutivos → investigar imediato (DT-### em [[../STATE]]).
- Timeout alto (> 30s) = sinal de teste mal desenhado; quebrar em passos menores ou usar `request` API direto pra setup.
- `waitForURL`, `waitForSelector` com timeout explícito; nunca `waitForTimeout` fixo.

---

## 7. CI pipeline

```yaml
jobs:
  e2e-web:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '22' }
      - run: bun install
      - run: bunx playwright install --with-deps chromium
      - run: bun run e2e                   # playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with: { name: e2e-trace, path: test-results/ }
```

Mobile similar com `flutter test integration_test/` em emulator via `reactivecircus/android-emulator-runner`.

---

## 8. Checklist E2E

- [ ] Cada jornada crítica tem ≥ 1 E2E?
- [ ] Setup via API (factory), não via UI?
- [ ] Cleanup pós-teste obrigatório?
- [ ] Dados sintéticos (nunca reais)?
- [ ] Sandbox de providers externos?
- [ ] Timeouts explícitos por passo?
- [ ] Screenshots/traces só em falha?
- [ ] Smoke test pós-deploy < 30s total?
- [ ] Flakiness documentado e corrigido dentro de 1 sprint?

---

## 9. Links

- [[_moc]]
- [[test-strategy]]
- [[integration]]
- [[load]]
- [[../01-domain/business-rules]]
- [[../00-product/portfolio-de-produtos]]
- [[../03-subprojects/_moc]]
