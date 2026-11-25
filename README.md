# Workflow de développement sécurisé assisté par IA

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
