# 14 novembre 2026

## Consigne 1 — Contrôle d'intégrité

- Identification du paquet SAT-ALPHA-3 dans `manifest.json` et calcul SHA-256 sur l’archive complète.
- Empreinte et taille différentes des valeurs annoncées : paquet non conforme au manifeste.
- **Flag validé :** `RM{65609e9a9e7710b8c1a1a9b2d56d7df2f3964eadf67edb2333103298c0e3eccf}`.
- **Détail :** [01_CONTROLE_INTEGRITE.md](01_CONTROLE_INTEGRITE.md).

## Consigne 2 — Analyse du paquet

- Extraction d’une copie de travail et contrôle avec `sha256sum -c SHA256SUMS` : dix fichiers conformes, seul `src/tcuplink/auth.py` non conforme.
- Identification de l’opérateur `0xA5` et du jeton `9e3ab7f04c118d26`, retour anticipé contournant registre, HMAC et anti-rejeu.
- **Flag validé :** `RM{src/tcuplink/auth.py:0xA5:9e3ab7f04c118d26}`.
- **Détail :** [02_ANALYSE_PAQUET.md](02_ANALYSE_PAQUET.md).

## Consigne 3 — Remontée à la source

- Consultation de l’historique du fichier et recherche des ajouts avec `git log -p -S '_MAINT_' src/tcuplink/auth.py`.
- Commit d’introduction : `2e192f32d090bf23007a6563b37c5c219e43c4a1` ; auteur déclaré : Camille Roussel, `c.roussel@segment-sol.aae`.
- **Flag :** `RM{2e192f32d090bf23007a6563b37c5c219e43c4a1:c.roussel@segment-sol.aae}`.
- **Détail :** [03_REMONTEE_SOURCE.md](03_REMONTEE_SOURCE.md).

## Consigne 4 — Identification d'IOC

- Affinage des règles YARA vers une acceptation `return True` placée avant le registre, la comparaison HMAC ou un rejet d’authentification.
- Cibles identifiées : GS-SUD, SAT-ALPHA-3, SAT-BETA-1, SAT-DELTA-2.
- **Flag validé :** `RM{GS-SUD:SAT-ALPHA-3:SAT-BETA-1:SAT-DELTA-2}`.
- **Détail :** [04_IDENTIFICATION_IOC.md](04_IDENTIFICATION_IOC.md).

## Consigne 5 — Confinement

- Repérage des sessions `7f3a9c` et `6e0b93` ouvertes avec `op=0xA5 auth=maint`, puis suivi par session malgré les changements d’opérateur.
- Six tâches à bloquer : `DK-8072`, `DK-4846`, `DK-8973`, `DK-7432`, `DK-2372`, `DK-1075`
- **Flag obtenu :** `RM{0301aeef7e2231e9a88be73a5dd34ffae8daaf9a}`.
- **Détail :** [05_CONFINEMENT.md](05_CONFINEMENT.md).

# 15 novembre 2026

## Consigne 6 — Attribution

- Identification du phishing `inbox-076.eml` : expéditeur `it-support@segrnent-sol.aae`, typosquatting et demande d’identifiants avec OTP. **Réception à 20:09:48 UTC le 5 novembre**.
- Corrélation avec `auth.ok` à 20:31:05 UTC ; tunnel `tun23` actif jusqu’à 23:48:19 UTC, couvrant la date d’auteur du commit à 21:14:06 UTC.
- **Flag final :** `RM{2026-11-05T20:31:05Z:it-support@segrnent-sol.aae}`.
- **Détail :** [06_ATTRIBUTION.md](06_ATTRIBUTION.md).
