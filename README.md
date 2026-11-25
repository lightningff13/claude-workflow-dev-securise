# Workflow de développement sécurisé assisté par IA

> **English summary**
>
> A Claude Code plugin that structures and secures AI-assisted application development. **The AI proposes, the senior developer decides.** The AI drafts specifications, executable mock-ups, the technical plan, code, tests and summaries. The senior developer approves at four gates: requirements and threat model, technical plan, code review, and production release. Automated checks that the AI cannot disable cover the rest.
>
> - **7 phases**: project setup, specification (user stories, mock-ups, STRIDE threat modeling), technical plan, implementation (TDD with the Transformation Priority Premise, small PRs), automated checks (lint, tests, accessibility, SAST, dependencies, secrets, SBOM), review (independent AI pre-review, then human review), pre-production.
> - **Model and effort per phase**: specialized sub-agents, with Opus for design and review and Sonnet for implementation, to balance cost, speed and quality.
> - **Context management**: an up-to-date project map, a log per feature and one session per phase, so the AI never re-reads the whole codebase.
> - **Lessons learned**: the AI proposes a lesson in each PR, and the senior developer decides whether to adopt it, observe it or reject it.
> - **Multi-platform support**: adaptations for Kotlin Multiplatform and Compose Multiplatform (Android, iOS, web, browser extension).
>
> - **Real-world example**: [ExpensesShared](https://github.com/lightningff13/ExpensesShared), a shared-expenses app on Android and web (Kotlin Multiplatform, Compose Multiplatform, Ktor), built with this plugin. See the "Exemple d'utilisation" section below.
>
> Install it with `/plugin marketplace add lightningff13/claude-workflow-dev-securise`, then `/plugin install workflow-dev-securise@claude-workflow-dev-securise`. The rest of this README and the skill itself are in French.

Plugin Claude Code qui structure et sécurise le développement d'applications avec l'IA.

**Principe : l'IA propose, le développeur senior décide.** L'IA rédige les spécifications, les maquettes, le plan, le code, les tests et les résumés. Le senior valide à quatre points de contrôle (gates), et des contrôles automatiques que l'IA ne peut pas désactiver couvrent le reste.

## Ce que fait le plugin

| Phase | Contenu | Validation |
|---|---|---|
| 0. Cadre | `CLAUDE.md`, carte du projet, sous-agents, garde-fous, CI, CODEOWNERS | Le senior relit chaque fichier |
| 1. Spécification | User stories, maquettes exécutables dans la stack cible, analyse des menaces STRIDE | Gate 1 |
| 2. Plan technique | Fichiers, découpage en PR, mesures de sécurité, ADR | Gate 2 |
| 3. Implémentation | TDD + TPP (Transformation Priority Premise), petites PR | — |
| 4. Contrôles automatiques | Lint, tests, accessibilité, SAST, dépendances, secrets, SBOM | CI bloquante |
| 5. Revue | Pré-revue par une IA relectrice indépendante, puis revue humaine | Gate 3 |
| 6. Pré-production | Staging, tests de bout en bout, tests dynamiques | Gate 4 |

S'y ajoutent :

- **Modèle et effort par phase** : Opus avec un effort élevé pour la conception et la revue, Sonnet avec un effort bas pour l'implémentation, via des sous-agents dédiés.
- **Gestion du contexte** : carte du projet tenue à jour, journal par fonctionnalité, une session par phase.
- **Leçons apprises** : l'IA propose une leçon dans chaque PR, le senior décide de l'intégrer, de l'observer ou de la rejeter.
- **Projets multi-plateformes** : adaptations pour Kotlin Multiplatform / Compose Multiplatform (Android, iOS, web, extension de navigateur).

## Exemple d'utilisation : ExpensesShared

[ExpensesShared](https://github.com/lightningff13/ExpensesShared) est un carnet de dépenses partagées pour groupes d'amis, sur Android et web avec la même parité (Kotlin Multiplatform, Compose Multiplatform, backend Ktor + PostgreSQL). Il est développé avec ce plugin, phase par phase.

**Phase 0 : cadre du projet** (branche `main`)
- [`CLAUDE.md`](https://github.com/lightningff13/ExpensesShared/blob/main/CLAUDE.md) à la racine et un par module (shared, composeApp, androidApp, server), avec la stack, les règles de sécurité et les transformations TPP adaptées à Kotlin.
- [Cinq sous-agents](https://github.com/lightningff13/ExpensesShared/tree/main/.claude/agents) : explorer, mockup-designer, implementer, critical-implementer, reviewer.
- [Carte du projet](https://github.com/lightningff13/ExpensesShared/blob/main/docs/architecture.md), [ADR](https://github.com/lightningff13/ExpensesShared/tree/main/docs/adr), modèles de demande et de STRIDE, CODEOWNERS, `SECURITY.md` et CI proposée.

**Phase 1 : spécification de la première fonctionnalité** (le squelette de l'application, Gate 1 approuvée le 2026-11-24)
- [Spec `01-skeleton.md`](https://github.com/lightningff13/ExpensesShared/blob/fc515e27e07b900c29ad77f7dcf75beeb8aa6ab1/docs/specs/01-skeleton.md) : 6 user stories avec critères Given/When/Then et parité Android / web, analyse STRIDE par périmètre (shared, serveur, Android, web, chaîne d'approvisionnement de la CI), décisions du senior et journal de la phase.
- [Maquettes exécutables](https://github.com/lightningff13/ExpensesShared/tree/fc515e27e07b900c29ad77f7dcf75beeb8aa6ab1/prototypes/01-skeleton) en Compose Multiplatform et [80 captures](https://github.com/lightningff13/ExpensesShared/tree/fc515e27e07b900c29ad77f7dcf75beeb8aa6ab1/docs/mockups/01-skeleton) : 10 états, thème clair et sombre, petit et grand écran, sur un vrai téléphone Android et dans Chrome (Wasm). Données fictives uniquement.

<p>
  <img src="https://raw.githubusercontent.com/lightningff13/ExpensesShared/fc515e27e07b900c29ad77f7dcf75beeb8aa6ab1/docs/mockups/01-skeleton/dummy-sign-in-filled-light-small-android.png" alt="Maquette du formulaire factice sur Android" width="220">
  &nbsp;&nbsp;
  <img src="https://raw.githubusercontent.com/lightningff13/ExpensesShared/fc515e27e07b900c29ad77f7dcf75beeb8aa6ab1/docs/mockups/01-skeleton/dummy-sign-in-filled-light-small-web.png" alt="Maquette du formulaire factice sur le web (Wasm)" width="220">
</p>

**Ce que la phase 1 a permis de détecter avant toute ligne de code de production :** sur le web en Compose Multiplatform Wasm, les champs du formulaire n'ont ni nom accessible ni type e-mail / mot de passe, et le focus du bouton est quasi invisible. Ces constats ont déclenché l'étude d'accessibilité prévue par l'ADR du projet, avant de valider ce choix technique pour le web.

## Installation

Dans Claude Code :

```
/plugin marketplace add lightningff13/claude-workflow-dev-securise
/plugin install workflow-dev-securise@claude-workflow-dev-securise
```

Puis redémarrez la session.

## Utilisation

1. Créez un dépôt vide et lancez Claude Code sur Opus avec un effort élevé :

   ```bash
   claude --model opus --effort high
   ```

2. Invoquez la skill et décrivez votre projet :

   ```
   /workflow-dev-securise:workflow-dev-securise-ia
   Nouveau projet : <ce que fait l'application, pour qui>.
   Stack : <…>. Plateformes : <…>.
   ```

3. **Phase 0** : l'IA vous montre chaque fichier du cadre avant de l'écrire. `.claude/settings.json` est créé en dernier, car il interdit ensuite à l'IA de modifier les règles, les sous-agents et la CI. Faites vous-même le premier commit.

4. Configurez sur GitHub ce que l'IA ne peut pas faire : protection de `main` (PR obligatoire, CI verte, approbation du CODEOWNER), détection de secrets, Dependabot, environnement de production protégé.

5. Redémarrez la session et vérifiez `/agents`, `/model` et `/effort`.

6. Pour chaque fonctionnalité : ouvrez une session, décrivez la demande, et validez à chaque gate en répondant « validé » ou en donnant vos objections. Entre deux phases, reprenez avec « Reprends la fonctionnalité <nom> ».

Conseil : commencez par une fonctionnalité « squelette » (l'application démarre sur chaque cible, CI verte, build de test) pour roder le workflow sans risque.

## Points d'attention

- Ne définissez pas `CLAUDE_CODE_EFFORT_LEVEL` ni `CLAUDE_CODE_SUBAGENT_MODEL` : ces variables priment sur les réglages des sous-agents et annulent la répartition par phase.
- Les règles de permission de `.claude/settings.json` peuvent évoluer avec les versions de Claude Code : relisez-les à l'installation.
- Le plugin guide l'IA ; il ne remplace ni la revue humaine ni un audit de sécurité.

## Licence

[MIT](LICENSE)
