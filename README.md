# interntrack-api

Une petite API pour suivre mes candidatures de stage, parce que mon tableau Excel commence à ne plus suffire.

## Pourquoi ce projet

Pour l'instant, je note tout dans un tableau : l'entreprise, le poste, la date à laquelle j'ai postulé et où j'en suis (envoyée, entretien, refus…). Ça marche au début, mais je ne sais jamais quand relancer, je perds le nom des personnes que j'ai rencontrées, et je dois rechercher le lien de l'offre à chaque fois. Et quand une entreprise a plusieurs offres, ça devient vite le bazar.

## Ce que je veux pouvoir faire

1. En tant qu'utilisateur, je veux enregistrer une entreprise afin de retrouver ses informations.
   → `POST /companies`
2. En tant qu'utilisateur, je veux voir toutes mes entreprises afin d'avoir une vue d'ensemble de là où je postule.
   → `GET /companies`
3. En tant qu'utilisateur, je veux enregistrer une candidature afin de garder le poste, la date et le lien de l'offre au même endroit.
   → `POST /applications`
4. En tant qu'utilisateur, je veux changer le statut d'une candidature afin de savoir où j'en suis avec chaque entreprise.
   → `PATCH /applications/{id}`
5. En tant qu'utilisateur, je veux voir les candidatures à relancer afin de ne plus en oublier.
   → `GET /applications?follow_up_before=...`
6. En tant qu'utilisateur, je veux ajouter un contact à une entreprise afin de savoir à qui écrire.
   → `POST /contacts`, `GET /contacts?company_id=...`

## Les endpoints

Pour chaque ressource, je reprends les 5 opérations de samedi : créer, lister, lire, modifier, supprimer.

**Entreprises**

```
POST   /companies
GET    /companies
GET    /companies/{id}
PATCH  /companies/{id}
DELETE /companies/{id}
```

**Candidatures**

```
POST   /applications
GET    /applications        (filtres : status, company_id, follow_up_before)
GET    /applications/{id}
PATCH  /applications/{id}
DELETE /applications/{id}
```

**Contacts**

```
POST   /contacts
GET    /contacts            (filtre : company_id)
GET    /contacts/{id}
PATCH  /contacts/{id}
DELETE /contacts/{id}
```

Les utilisateurs n'ont pas encore leurs propres routes. Ils serviront surtout pour la connexion, en semaine 5.

## Le modèle de données

Quatre tables : `users`, `companies`, `applications` et `contacts`. Le schéma complet est dans [docs/schema.dbml](docs/schema.dbml) (il s'ouvre sur [dbdiagram.io](https://dbdiagram.io)).

```
users ──< companies ──< applications
                    └─< contacts
```

Comment je vois les relations :

- **Un utilisateur a plusieurs entreprises**, mais chaque entreprise n'appartient qu'à un seul utilisateur. C'est ce qui permettra plus tard que chacun ne voie que ses propres données.
- **Une entreprise peut avoir plusieurs candidatures** : par exemple deux offres différentes chez le même employeur, ou une nouvelle tentative l'année suivante.
- **Un contact appartient à une entreprise.** Pour savoir à qui il est, on passe par l'entreprise : le contact appartient à l'utilisateur qui possède l'entreprise. Pareil pour les candidatures. Comme ça, je ne stocke pas deux fois la même information.

Toutes les tables ont un `id` et un `created_at`. Pour une candidature, je garde :

- `position` : l'intitulé du poste
- `status` : `to_apply`, `applied`, `interview`, `offer` ou `rejected`
- `applied_at` : la date d'envoi (vide si je n'ai pas encore postulé)
- `follow_up_at` : la date à laquelle relancer
- `offer_url` : le lien de l'offre
- `notes` : tout le reste
