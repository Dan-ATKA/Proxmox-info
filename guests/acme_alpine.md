# Configuration ACME minimaliste sur Alpine

Oui, et c’est là que ton intuition est excellente : **Alpine fournit déjà exactement ce que tu cherches**, c’est‑à‑dire un système ACME minimaliste, géré par OpenRC, avec un utilisateur dédié, un répertoire dédié, et très peu de customisation à faire.

En fait, Alpine a déjà fait 80 % du travail pour toi.

Je vais te montrer ce qui existe *clé en main*, et comment l’utiliser pour rester dans ton modèle “immutable + auto‑renouvelant + minimal”.

---

## 1) Le paquet Alpine que tu cherches : **`acme-client`**

Quand tu installes :

```shell
apk add acme-client
```

Alpine fournit automatiquement :

* un **service OpenRC** : `/etc/init.d/acme-client`
* un **fichier de configuration** : `/etc/`[`acme-client.conf`](https://acme-client.conf)
* un **répertoire dédié** : `/etc/acme`
* un **utilisateur système dédié** : `acme`
* des **permissions déjà durcies**
* un **cron OpenRC** pour le renouvellement automatique

C’est littéralement ce que tu décrivais.

Tu n’as presque rien à inventer.

---

## 2) Ce que fait Alpine automatiquement (et proprement)

Quand tu installes `acme-client`, Alpine crée :

### ✔️ Un utilisateur dédié

```bash
acme:x:100:101:ACME client:/var/empty:/sbin/nologin
```

* pas de shell
* pas de home
* pas de droits d’écriture ailleurs

### ✔️ Un répertoire dédié

```
/etc/acme
```

Avec permissions strictes.

### ✔️ Un service OpenRC

```
/etc/init.d/acme-client
```


Ce service :

* tourne sous l’utilisateur `acme`
* ne dépend que du réseau
* ne touche à rien d’autre que `/etc/acme`

### ✔️ Un cron OpenRC automatique

```
/etc/periodic/daily/acme-client
```


Il lance le renouvellement sans que tu aies à écrire un script.

---

## 3) Ce que tu dois faire toi-même (et c’est minime)

### 1) Configurer `/etc/`[`acme-client.conf`](https://acme-client.conf)

Exemple minimal :

```
authority letsencrypt {
    api url "https://acme-v02.api.letsencrypt.org/directory"
    account key "/etc/acme/letsencrypt-privkey.pem"
}

domain "site1.example.com" {
    domain key "/etc/acme/site1.key"
    domain fullchain "/etc/acme/site1.fullchain"
    sign with letsencrypt
    challenge http "/var/www/site1/.well-known/acme-challenge"
    post-hook "/usr/bin/kill -HUP $(pidof lighttpd)"
}
```


### 2) Donner à lighttpd accès en lecture aux certificats

Tu peux faire :

```
chgrp lighttpd /etc/acme
chmod 750 /etc/acme
```


Ou copier les certificats dans `/etc/lighttpd/certs` via un hook.

### 3) Activer le service

```
rc-update add acme-client default
rc-service acme-client start
```


Et c’est tout.

---

## 4) Pourquoi c’est parfait pour ton modèle “immutable”

Tu obtiens :

* un système **minimaliste**
* un renouvellement **automatique**
* un utilisateur **isolé**
* un répertoire **isolé**
* un service **OpenRC natif**
* aucune dépendance lourde
* aucune logique custom
* une migration future **triviale** (Alpine → Alpine, ou Alpine → autre)

Tu restes dans la philosophie :

> “Le serveur doit pouvoir survivre même si je l’abandonne.”

Et tu ajoutes HTTPS sans casser ton modèle.

---

## 5) Pourquoi `acme-client` est mieux qu’[acme.sh](https://acme.sh) dans ton cas

* plus petit
* plus simple
* plus audité
* intégré nativement à Alpine
* service OpenRC fourni
* pas de scripts shell massifs
* pas de dépendances externes
* plus proche de ton idéal “immutable”

[acme.sh](https://acme.sh) est plus flexible, mais `acme-client` est plus “systémique”, plus propre, plus Alpine.

---

## Conclusion

👉 Oui, Alpine fournit déjà un système ACME **clé en main**, minimaliste, sécurisé, OpenRC‑friendly, et parfaitement adapté à ton modèle d’immutabilité.

👉 Tu n’as presque rien à customiser : juste la conf du domaine et le hook pour lighttpd.

👉 Tu restes dans un modèle simple, durable, migrable, et très robuste.

---
