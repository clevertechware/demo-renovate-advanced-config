# demo-renovate-advanced-config

Presets Renovate partagés et workflow Renovate unique des dépôts `clevertechware/demo-renovate-advanced-*`. Dépôt de démo
de la série sur Renovate.

## Hiérarchie des presets

```mermaid
graph TD
  config:recommended --> global
  docker-images --> global
  github-actions --> global
  global --> java-base
  global --> go-base
  global --> infra
  java-base --> java-service
  go-base --> go-service
```

Une flèche va d'un preset vers celui qui l'étend ou l'inclut.

| Preset | Contenu |
|---|---|
| `default` | `enabled: false`. Renovate ne l'applique automatiquement qu'à un dépôt nommé `renovate-config`. |
| `global` | `config:recommended`, `docker-images`, `github-actions`, dashboard, `minimumReleaseAge` avec `internalChecksFilter: strict`, `gitIgnoredAuthors`, limites de PR. |
| `docker-images` | Pas d'âge minimal pour les datasources `docker` et `java-version` ; fusion automatique des minor et patch des images Docker ; majeures de `postgres` soumises à approbation. |
| `github-actions` | Fusion automatique des mises à jour minor et patch des actions (`matchDepTypes: ["action"]`). |
| `java-base` | Étend `global` ; fusion automatique des minor et patch Maven (`maven`, `maven-wrapper`) ; majeure de `spring-boot-starter-parent` soumise à approbation. |
| `go-base` | Étend `global` ; `gomodTidy`, `gomodUpdateImportPaths` ; groupe `bun` ; fusion automatique des minor et patch des modules (`gomod`). |
| `infra` | Étend `global`. |
| `java-service`, `go-service` | Étendent `java-base` et `go-base` ; point d'entrée des dépôts de service. |

## Brancher un dépôt

Ajouter `.github/renovate.json` au dépôt, sans workflow Renovate local :

```json
{
  "extends": ["github>clevertechware/demo-renovate-advanced-config//infra"]
}
```

Le nom du dépôt doit correspondre à `clevertechware/demo-renovate-advanced-*` (filtre de découverte) et la GitHub App Renovate
doit y être installée. `RENOVATE_ONBOARDING=false` garantit qu'un dépôt du filtre sans `renovate.json` est ignoré au lieu de recevoir une PR d'onboarding.

## Workflows

- `validate.yml` : exécute `renovate-config-validator --strict --no-global` sur tous les presets, à chaque push et pull request.
- `renovate.yml` : passage nocturne (02:00 UTC) ou manuel sur tous les dépôts découverts. Requiert la variable
  `RENOVATE_APP_CLIENT_ID` et le secret `RENOVATE_APP_PRIVATE_KEY`. Il échoue tant que l'App n'est pas créée.

`RENOVATE_AUTODISCOVER_FILTER` est un motif glob. Plusieurs filtres se séparent par une virgule, donc le motif lui-même
n'en contient pas.

## Valider en local

```bash
npx --yes --package renovate@44 -- renovate-config-validator --strict --no-global *.json
```

`--no-global` valide les fichiers comme configuration de dépôt, ce qui rejette une option réservée à l'auto-hébergement
(`autodiscover`, par exemple), y compris dans un preset.

## Âge minimal des versions

`global` fixe `minimumReleaseAge` à 3 jours. Le preset `docker-images` le désactive pour les datasources `docker` et
`java-version` : avec Renovate 44.225.2, les versions des images `postgres`, `golang` et `eclipse-temurin` arrivent sans date
de publication, comme celles de `java-version`, et restent en attente. Les tags des images officielles sont aussi repoussés à
chaque reconstruction, ce qui relance le délai. `config:recommended` limite déjà `eclipse-temurin` aux versions LTS
(`workarounds:javaLTSVersions`).
