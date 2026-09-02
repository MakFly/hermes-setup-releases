# hermes-setup — releases

Binaires statiques de `hermes-setup` (linux/amd64, linux/arm64) et leur
`SHA256SUMS`. Le code source vit dans un dépôt privé ; ce dépôt ne contient
que les artefacts publiés par `make publish`.

```bash
curl -fsSLO https://github.com/MakFly/hermes-setup-releases/releases/latest/download/hermes-setup_linux_amd64
curl -fsSLO https://github.com/MakFly/hermes-setup-releases/releases/latest/download/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing
sudo install -m 0755 hermes-setup_linux_amd64 /usr/local/sbin/hermes-setup
```

Un hôte déjà installé se met à jour avec `hermes-setup upgrade --yes`
(vérification SHA-256, redémarrage des unités).
