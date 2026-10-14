---
name: commit
description: Rédiger et créer un commit au format Conventional Commits pour RééducAssist. À utiliser chaque fois qu'un ticket est terminé ou qu'un commit doit être fait.
---

# Commit au format Conventional Commits

## Quand committer
- Quand le ticket (ou une étape cohérente du ticket) est terminé : critères remplis, tests au vert, diff relu.
- Jamais sur `main` : toujours sur la branche du ticket.

## Étapes
1. Lancer `git status` et `git diff --staged` pour voir exactement ce qui part dans le commit.
2. Si le diff mélange plusieurs sujets, proposer de le découper en plusieurs commits.
3. Choisir le type : feat, fix, test, docs, refactor, perf, chore, ci, build.
4. Choisir la portée selon les dossiers modifiés : backend, ai-service, frontend, infra, docs, ci, claude.
5. Rédiger la première ligne : `type(portée): description` en français, à l'impératif, sans majuscule initiale ni point final, 72 caractères au maximum.
6. Ajouter un corps si le « pourquoi » n'est pas évident, puis le pied `Refs: RA-xxx`.
7. Présenter le message au développeur et expliquer le choix du type et de la portée.
8. Lancer `git commit`.

## Si le hook refuse le commit
- Lire le message d'erreur, corriger le message et recommencer.
- Ne jamais utiliser `--no-verify` ni `-n` pour contourner le hook.

## Exemples
- `feat(backend): créer le modèle Patient pseudonymisé`
- `test(ai-service): ajouter les cas d'injection de prompt`
- `fix(frontend): corriger l'affichage des sources sur mobile`
- `docs: ajouter l'ADR-005 sur la Row-Level Security`
- `chore(claude): ajouter le skill commit`
- `ci: vérifier le titre des pull requests`
