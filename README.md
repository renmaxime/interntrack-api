# interntrack-api

API REST pour suivre ses candidatures de stage : entreprises, candidatures et contacts.

## Problème

Aujourd'hui, je suis mes candidatures dans un tableau : le nom de l'entreprise, le poste, la date d'envoi et le statut (envoyée, entretien, refus…). Il me manque une date de relance pour savoir qui recontacter et quand, un endroit pour noter les personnes rencontrées dans chaque entreprise, et le lien vers l'offre. Quand une même entreprise publie plusieurs offres, le tableau devient vite illisible.

## User stories

1. En tant qu'utilisateur, je veux enregistrer une entreprise afin de retrouver ses informations (site, ville, secteur).
2. En tant qu'utilisateur, je veux voir la liste de mes entreprises afin d'avoir une vue d'ensemble de mes cibles.
3. En tant qu'utilisateur, je veux enregistrer une candidature pour une entreprise afin de garder le poste, la date d'envoi et le lien de l'offre.
4. En tant qu'utilisateur, je veux mettre à jour le statut d'une candidature afin de savoir où j'en suis dans chaque processus.
5. En tant qu'utilisateur, je veux lister mes candidatures filtrées par statut ou par date de relance afin de savoir qui relancer cette semaine.
6. En tant qu'utilisateur, je veux ajouter un contact à une entreprise afin de me souvenir à qui écrire et comment le joindre.

| Story | Endpoints |
|-------|-----------|
| 1 | `POST /companies` |
| 2 | `GET /companies` |
| 3 | `POST /applications` |
| 4 | `PATCH /applications/{id}` |
| 5 | `GET /applications?status=…&follow_up_before=…` |
| 6 | `POST /contacts`, `GET /contacts?company_id=…` |

## Endpoints prévus

Chaque ressource expose les 5 opérations CRUD : créer, lister, lire un élément, modifier, supprimer.

### Companies

- `POST /companies` : créer une entreprise
- `GET /companies` : lister mes entreprises
- `GET /companies/{id}` : détail d'une entreprise
- `PATCH /companies/{id}` : modifier une entreprise
- `DELETE /companies/{id}` : supprimer une entreprise

### Applications

- `POST /applications` : créer une candidature (avec `company_id`)
- `GET /applications` : lister mes candidatures (filtres `status`, `company_id`, `follow_up_before`)
- `GET /applications/{id}` : détail d'une candidature
- `PATCH /applications/{id}` : modifier une candidature (statut, date de relance…)
- `DELETE /applications/{id}` : supprimer une candidature

### Contacts

- `POST /contacts` : créer un contact (avec `company_id`)
- `GET /contacts` : lister mes contacts (filtre `company_id`)
- `GET /contacts/{id}` : détail d'un contact
- `PATCH /contacts/{id}` : modifier un contact
- `DELETE /contacts/{id}` : supprimer un contact

### Users

Pas de CRUD complet pour l'instant : les utilisateurs serviront à l'authentification et à l'isolation des données (semaine 5).

## Modèle de données

Schéma complet : [docs/schema.dbml](docs/schema.dbml), à coller dans [dbdiagram.io](https://dbdiagram.io).

```
users 1 ──< companies 1 ──< applications
                       1 ──< contacts
```

- **users → companies (1-N)** : un utilisateur a plusieurs entreprises, et chaque entreprise appartient à un seul utilisateur (`companies.user_id`). C'est la base de l'isolation des données.
- **companies → applications (1-N)** : une entreprise peut avoir plusieurs candidatures (plusieurs offres, ou une nouvelle candidature l'année suivante). Une candidature concerne une seule entreprise (`applications.company_id`).
- **companies → contacts (1-N)** : un contact appartient à une entreprise (`contacts.company_id`). Son propriétaire est l'utilisateur de cette entreprise : on le retrouve par `contacts.company_id → companies.user_id`, sans dupliquer de `user_id` dans la table. C'est aussi le cas pour les candidatures.

### Colonnes d'une candidature

| Colonne | Type | Rôle |
|---------|------|------|
| `id` | integer, PK | identifiant |
| `company_id` | integer, FK → companies | entreprise concernée |
| `position` | varchar | intitulé du poste |
| `status` | enum | `to_apply`, `applied`, `interview`, `offer`, `rejected` |
| `applied_at` | date, nullable | date d'envoi (vide si pas encore envoyée) |
| `follow_up_at` | date, nullable | date de relance prévue |
| `offer_url` | varchar, nullable | lien de l'offre |
| `notes` | text, nullable | notes libres |
| `created_at` | timestamp | date de création de la ligne |
