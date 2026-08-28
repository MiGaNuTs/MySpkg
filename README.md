# gitsyncpkg — v0 (validation du pipeline)

Cette v0 ne fait qu'une chose : afficher "Hello World" + son numéro de version
sur un mini-serveur HTTP (port 8877), pour valider tout le pipeline de
distribution DSM avant d'écrire la vraie logique métier.

## Mise en place (une fois)

1. Crée un repo GitHub public, ex. `MiGaNuTs/gitsyncpkg`, pousse ce contenu.
2. Dans **Settings → Pages**, source = **GitHub Actions** (pas "branch").
3. Vérifie que **Settings → Actions → General → Workflow permissions**
   autorise "Read and write permissions" (nécessaire pour créer la Release).

## Plan de test

1. **Ajout de la source** : sur le DS216j, Package Center → Paramètres →
   Source de paquets → Ajouter → colle l'URL Pages
   (`https://<user>.github.io/<repo>/packages.json`).
2. **Tag + install depuis GitHub** :
   ```
   git tag v0.1.0
   git push origin v0.1.0
   ```
   Attends que l'Action termine (Release + Pages publiés), puis dans
   Package Center, cherche "Git Sync" dans le catalogue et installe-le.
3. **Vérification** : ouvre `http://<ip-du-216j>:8877/` (ou le bouton
   "Ouvrir" dans Package Center) → doit afficher "Hello World" + version 0.1.0.
4. **Nouvelle version** :
   ```
   git tag v0.1.1
   git push origin v0.1.1
   ```
   Attends la fin de l'Action, puis dans Package Center, force un refresh
   de la source si la puce de mise à jour n'apparaît pas immédiatement
   (retirer/rajouter la source si besoin).
5. **Mise à jour** : clique sur "Mettre à jour" dans Package Center →
   revérifie `http://<ip-du-216j>:8877/` → doit maintenant afficher 0.1.1.

## Structure

```
spk-repo/
├── INFO                  # métadonnées du package (version bumpée par la CI)
├── scripts/
│   ├── start-stop-status # lance/stoppe busybox httpd sur le port 8877
│   ├── postinst
│   └── preuninst
└── src/
    └── www/
        └── index.html    # page hello world (version injectée par la CI)
```

Le `.spk` et `packages.json` sont générés automatiquement par
`.github/workflows/release.yml` à chaque tag `vX.Y.Z` — rien à construire
à la main.
