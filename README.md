# RééducAssist

> ⚠️ **Avertissement**
> RééducAssist est un **démonstrateur** réalisé dans un cadre d'apprentissage.
> Il fonctionne uniquement avec des **données synthétiques** : aucune vraie donnée de santé ne doit y être saisie.
> Ce n'est **pas un dispositif médical** et il ne remplace en aucun cas l'avis d'un professionnel de santé.

## Structure du monorepo

| Dossier | Contenu |
|---|---|
| `backend/` | API principale (Python) |
| `ai-service/` | Service d'assistant IA |
| `frontend/` | Interface React |
| `infra/` | Docker, déploiement |
| `docs/` | Conception et documentation du projet |
| `.github/workflows/` | Intégration continue (GitHub Actions) |

## Règles de contribution

- Aucun commit direct sur `main` : chaque ticket suit le cycle branche, commits, Pull Request, CI verte, merge.
- Messages de commit au format [Conventional Commits](https://www.conventionalcommits.org/fr/) : `type(portée): description`.
- Aucun secret dans le code : la configuration passe par des variables d'environnement.

## Licence

[MIT](LICENSE)
