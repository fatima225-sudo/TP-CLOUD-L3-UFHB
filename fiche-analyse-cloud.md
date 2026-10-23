# Cartographier et chiffrer le cloud : fiche d'analyse

**Cours :** Cloud computing, Licence 3 RIST / BDGL Maths-Info, Université Félix Houphouët-Boigny

**Livrable :** TP de séance (1 h 15, binôme) — cartographier et chiffrer le cloud

**Nom du professeur :** Bakayoko Ibrahima

**Membres :** Cissé Fatim — Douhou Deborah Eliane — Kri Banthi Hermes

---

## Partie A. Cartographie de 8 services du quotidien

Légende NIST : **LS** = libre-service à la demande, **RÉS** = accès réseau étendu, **MUT** = mise en commun des ressources, **ÉLA** = élasticité rapide, **MES** = service mesuré.

| #   | Service                          | Modèle | LS  | RÉS | MUT | ÉLA | MES | Hébergeur probable             |
| --- | -------------------------------- | ------ | --- | --- | --- | --- | --- | ------------------------------ |
| 1   | Gmail                            | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | Google (infrastructure propre) |
| 2   | Google Docs                      | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | Google (infrastructure propre) |
| 3   | Netflix                          | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | AWS                            |
| 4   | Dropbox                          | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | Infrastructure propre + AWS    |
| 5   | Wave (mobile money)              | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | à vérifier                     |
| 6   | Vercel                           | PaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | AWS                            |
| 7   | Google Drive                     | SaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | Google (infrastructure propre) |
| 8   | Machine virtuelle EC2 / Azure VM | IaaS   | ✔   | ✔   | ✔   | ✔   | ✔   | AWS / Azure                    |

**Méthode pour deviner l'hébergeur :** recherche DNS (`nslookup`), recherche de l'adresse IP sur un site « IP lookup », blog technique de l'entreprise, offres d'emploi mentionnant la stack.

**Justification du classement (exemples) :**

- **SaaS :** l'utilisateur ne gère rien (ni serveur, ni système, ni application). Il utilise le logiciel via un navigateur ou une application (Gmail, Docs, Netflix).
- **PaaS :** le développeur déploie son code, la plateforme gère serveurs et système (Vercel).
- **IaaS :** on loue une machine virtuelle et on gère le système et les logiciels soi-même (EC2).

**Remarques sur les 5 caractéristiques NIST :**

Tous les services listés cochent les 5 critères NIST, mais à des degrés différents. Le « service mesuré » est visible via l'abonnement (Netflix) ou le quota de stockage (Drive). L'élasticité est invisible pour l'utilisateur mais réelle côté fournisseur. Pour un service comme Wave, le caractère « libre-service » est plus marqué (inscription immédiate sans intervention humaine) que pour un service comme EC2, où l'utilisateur doit configurer lui-même sa machine.

---

## Partie B. Chiffrage : cloud vs serveur physique

### 1. Estimation cloud (calculateur AWS ou Azure)

| Élément | Configuration | Coût mensuel |
| ------- | ------------- | ------------ |
| Machine virtuelle | 2 vCPU, 4 Go RAM, Linux, région | X $ |
| Stockage | 50 Go (disque SSD) | Y $ |
| **Total mensuel** |  | **(X + Y) $** |
| **Total annuel** |  | **((X + Y) × 12) $** |

Capture d'écran du calculateur à insérer ici.

### 2. Estimation serveur physique

| Élément | Coût |
| ------- | ---- |
| Achat d'un serveur d'entrée de gamme | A $ |
| Électricité (par mois) | B $ |
| Maintenance / administration (par mois) | C $ |
| Durée d'amortissement | 4 ans (48 mois) |
| **Coût mensuel équivalent** | **(A / 48 + B + C) $** |

### 3. Comparaison

| Critère | Cloud | Serveur physique |
| ------- | ----- | ---------------- |
| Coût de départ | Nul | Élevé |
| Coût mensuel | (X + Y) $ | (A / 48 + B + C) $ |
| Coût sur 4 ans | ((X + Y) × 48) $ | (A + (B + C) × 48) $ |
| Flexibilité | Très forte | Faible |
| Gestion du matériel | Fournisseur | Nous |

### 4. Conclusion : quand le cloud est-il rentable ?

Le cloud est avantageux quand :

- la charge est **variable ou imprévisible** (on paie à l'usage, on ajuste vite) ;
- on **démarre un projet** et qu'on ne veut pas immobiliser un investissement ;
- on n'a **pas d'équipe** pour gérer le matériel, l'électricité, les pannes et la sécurité physique.

Le serveur physique devient plus intéressant quand :

- la charge est **stable et élevée** sur plusieurs années ;
- on a déjà les compétences et le local adaptés.

**Notre conclusion :** pour notre cas (petit projet, étudiants, startup), le cloud est rentable ou moins rentable parce que la raison appuyée par vos chiffres. Au-delà d'environ N mois d'utilisation continue, le serveur physique reviendrait moins cher, à condition de compter aussi le coût du personnel.