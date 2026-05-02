---
title: ONB-S4 — Upload Ata de Posse (Mobile, Síndico)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, onboarding, sindico]
project: master-sindico
persona: sindico
screen_id: ONB-S4
sub_produto: governanca-institucional
plan_requirement: trial
status: specification
stack: mobile
created: 2026-04-24
---

# ONB-S4 — Upload Ata de Posse (Mobile, Síndico)

## Finalidade

Upload do documento oficial de posse (PDF ou imagem) para análise KYB síndico.

## Persona e ABAC

- Síndico em onboarding.

## Fluxo mobile

1. Picker: câmera (foto doc) ou arquivo PDF (`file_picker`).
2. Preview antes upload.
3. `POST /storage/presign` → PUT S3 → `PATCH onboarding/sessions/me { ata_of_posse_key }`.
4. Compressão client-side (img max 2MB).
5. Próximo ONB-S5.

## Widget Flutter principal

`OnbS4UploadPage` com `image_picker` + `file_picker` + `Image.file` preview.

## Platform-specific notes

- **Permissions**: camera + storage (Android < 10).
- `NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription`.
- Progress indicator durante upload.

## Estados

- Empty / Picked / Uploading(%) / Uploaded / Error.

## Offline behavior

- Upload exige rede; picker livre offline + upload na fila sync.

## Push notification trigger

Nenhum.

## Links

- [[_moc]] · [[ONB-S3]] · [[ONB-S5]]
