# Orchestrateur ou integration continue ? (TP06)

| Critere | Prefect (orchestrateur) | GitHub Actions (CI) |
|---------|-------------------------|---------------------|
| Ce qui declenche l'execution | La planification cron (chaque lundi a 6 h), un Quick run depuis l'interface, ou `python flow_mlops.py` | Chaque `git push`, ou le bouton "Run workflow" (workflow_dispatch) |
| Ou s'execute le traitement | Sur notre poste : processus `serve` ou worker du work pool `pool-tp`, avec les donnees DVC locales | Sur une machine Linux temporaire de GitHub (ubuntu-latest), donnees v2 regenerees |
| Qui surveille, et ou | Le serveur Prefect : interface http://127.0.0.1:4200 (Runs, Deployments, Work Pools), etat, duree et journaux de chaque tache | GitHub : onglet Actions, journal de chaque etape, commit marque vert ou rouge, e-mail d'echec |
| Ce qui se passe en cas d'echec | La tache et le flow passent en Failed ; `notifier` ne s'execute pas ; `rejouer_pipeline` est d'abord relancee 2 fois | L'etape renvoie le code 1, le job devient rouge, le commit est marque en echec ; les rapports sont conserves (`if: always()`) |
| Type de tache adapte | Traitements recurrents de production : reentrainement planifie, batch, enchainements longs a surveiller | Verification de chaque modification du code : tests, reproductibilite, regle de qualite avant fusion |
| Ce qui est versionne dans Git | `flow_mlops.py`, `planifier.py`, `prefect.yaml` (l'historique des runs reste dans la base Prefect) | `ci.yml`, `verifier_qualite.py`, `requirements-ci.txt` (l'historique des runs est garde par GitHub) |

## Conclusion (5 lignes)

Prefect et GitHub Actions automatisent tous les deux le pipeline, mais a des moments differents.
GitHub Actions valide le code a chaque modification : un changement qui degrade le modele (F1 = 0.25) est bloque et visible sur le commit.
Prefect execute et surveille les traitements recurrents : reentrainement du lundi, etat et journaux par tache, arret automatique sous le seuil.
La meme regle de qualite (seuil_f1 = 0.82 de params.yaml) est appliquee des deux cotes.
Les deux outils sont complementaires : aucun ne remplace l'autre.
