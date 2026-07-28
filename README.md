# 🍻 Est-ce l'heure de l'apéro ?

Page statique mono-fichier qui répond à la seule question qui compte vraiment.
Sert de bac à sable pour tester une chaîne **build → registry → runtime conteneurisé**.

![static](https://img.shields.io/badge/type-static-blue)
![nginx](https://img.shields.io/badge/runtime-nginx--unprivileged-green)
![deps](https://img.shields.io/badge/dependencies-0-brightgreen)

---

## ✨ Caractéristiques

- **Un seul fichier** : `index.html` (~64 Ko), CSS et JS inline
- **Zéro requête externe** : police Lato (woff2, subset latin) et illustration SVG embarquées en base64
  → aucun CDN, aucun tiers, RGPD-friendly, fonctionne en air-gapped
- **Mobile first**, responsive, sans framework
- **Rootless** : image `nginx-unprivileged`, prête pour `runAsNonRoot: true`

## 🕐 Règles métier

| Créneau (heure locale du navigateur) | Réponse |
|---|---|
| 11:00 → 13:30 | *Bien sûr! A la tienne!* 🍻 |
| 17:30 → 21:00 | *Bien sûr! A la tienne!* 🍻 |
| Le reste du temps | *Tu dois attendre un peu!...* |

Le fuseau horaire est celui du client. Réévaluation automatique toutes les 60 s.

## 🧪 Mode test

Paramètre `?tm=HHMM` pour simuler une heure et valider les deux cas à tout moment :

```
index.html?tm=1200   → 🍻
index.html?tm=1400   → attendre
index.html?tm=1830   → 🍻
index.html?tm=2500   → invalide, ignoré (fallback heure réelle)
```

Validation stricte : exactement 4 chiffres, `00 ≤ HH ≤ 23`, `00 ≤ MM ≤ 59`.
Toute valeur non conforme est ignorée silencieusement (fail-safe).
Un badge sous la carte indique l'heure retenue et signale explicitement le mode test.

## 🐳 Build & run

```bash
docker build -t apero:1.0.0 .
docker run --rm -p 8080:8080 apero:1.0.0
# → http://localhost:8080
```

> Le conteneur écoute sur **8080** (image unprivileged, pas de bind sur port < 1024).

### Push vers GCP Artifact Registry

```bash
gcloud auth configure-docker europe-west1-docker.pkg.dev

# Build + push côté Google (recommandé, évite les aléas réseau locaux)
gcloud builds submit \
  --tag europe-west1-docker.pkg.dev/$PROJECT_ID/sites/apero:1.0.0 .
```

### Déploiement Cloud Run

```bash
gcloud run deploy apero \
  --image europe-west1-docker.pkg.dev/$PROJECT_ID/sites/apero:1.0.0 \
  --port 8080 --allow-unauthenticated --region europe-west1
```

## 📁 Structure

```
.
├── index.html    # l'application complète
├── Dockerfile    # nginx-unprivileged:alpine-slim + healthcheck
└── README.md
```

## 🗺️ Idées d'évolution

- [ ] Extraire `estApero(minutes)` en fonction pure + tests unitaires (FIRST)
- [ ] `cloudbuild.yaml` + trigger GitHub, tag par `$SHORT_SHA`
- [ ] Manifests Kubernetes (Deployment / Service / Ingress) pour le lab GKE
- [ ] Créneaux configurables au build

## 📄 Licence

MIT — l'illustration SVG est une création originale, libre de droits.