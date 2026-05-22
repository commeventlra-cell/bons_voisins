<<<<<<< HEAD
# Contacts La Résidence

V1 fonctionnelle d’une web-app responsive pour centraliser les contacts consentants de La Résidence, gérer les programmes Bons Voisins et Fidélité, suivre les demandes, filtrer les contacts et exporter des listes WhatsApp ciblées.

## Stack

- Frontend : React + Vite
- Styling : Tailwind CSS
- Backend : Node.js + Express
- Base locale : fichier JSON persistant, sans dépendance native ni `node-gyp`
- Auth admin : email + mot de passe, session JWT

## Installation

```bash
npm.cmd install
```

Copier le fichier d’environnement :

```bash
cp .env.example .env
```

Sur Windows PowerShell :

```powershell
Copy-Item .env.example .env
```

Variables principales :

```env
PORT=3000
JWT_SECRET=changez-moi-en-production
ADMIN_EMAIL=admin@laresidence.mg
ADMIN_PASSWORD=admin123
VITE_RESIDENCE_WHATSAPP_NUMBER=261340000000
```

`VITE_RESIDENCE_WHATSAPP_NUMBER` doit contenir le numéro WhatsApp international de La Résidence, sans espaces. Il sert à générer les liens `https://wa.me/...` dans l’admin.

## Lancement

Initialiser la base et l’utilisateur admin :

```bash
npm.cmd run seed
```

Lancer en développement :

```bash
npm.cmd run dev
```

Frontend : `http://localhost:5173`

Backend API : `http://localhost:3000`

La commande affiche aussi ces deux adresses au démarrage.

Si vous préférez lancer les deux parties dans deux terminaux séparés :

```bash
npm.cmd run dev:server
npm.cmd run dev:client
```

Le serveur Express de développement est lancé sans mode `node --watch` afin d’éviter les erreurs Windows `spawn EPERM` sur certaines installations.

Build production :

```bash
npm.cmd run build
npm.cmd run start
```

## Routes publiques

- `/bons-voisins`
- `/fidelite`
- `/demande/brunch`
- `/demande/soiree`
- `/demande/moments-conviviaux`
- `/demande/piscine`
- `/demande/anniversaire`
- `/demande/chambre`
- `/demande/seminaire`
- `/demande/reception`
- `/demande/pizza`
- `/demande/plat-du-jour`
- `/demande/menu-des-voisins`

Les formulaires `/demande/:type` intègrent les choix Bons Voisins, Fidélité et offres dans le même envoi. Le client ne doit pas revenir vers un autre formulaire après confirmation.

## Dates de brunch

Le formulaire `/demande/brunch` propose automatiquement les 4 prochaines dates possibles.

Configuration optionnelle :

```text
src/config/eventDates.js
```

Ajoutez des dates au format ISO si vous voulez imposer des dates précises :

```js
export const brunchDates = ['2026-05-24', '2026-06-07'];
```

Seules les dates supérieures ou égales à la date du jour sont affichées. Si aucune date future n’est configurée, l’app génère automatiquement les 4 prochains dimanches. L’option `Autre date / demande spéciale` reste toujours disponible.

## Espace admin

- `/admin/login`
- `/admin`
- `/admin/contacts`
- `/admin/contacts/nouveau`
- `/admin/contacts/:id`
- `/admin/imports/sondage-moments-conviviaux`
- `/admin/exports`
- `/admin/liens-whatsapp`
- `/admin/offres`

## Liens WhatsApp préremplis

La page `/admin/liens-whatsapp` génère des liens prêts à copier pour les publications : Bons Voisins, Fidélité, Plat du jour, Pizza, Brunch, Soirée, Piscine, Anniversaire, Chambre, Séminaire, Réception, Moments conviviaux et Menu des Voisins.

Les messages et le numéro par défaut sont configurés dans :

```text
src/config/whatsapp.js
```

Compte par défaut après `npm.cmd run seed` :

- Email : `admin@laresidence.mg`
- Mot de passe : `admin123`

Changez ces valeurs dans `.env` avant usage réel.

## Données

Les données locales sont créées dans :

```text
data/contacts-la-residence.json
```

Une sauvegarde simple peut aussi être créée dans :

```text
data/contacts-la-residence.json.bak
```

La V1 écrit directement dans le fichier JSON avec quelques retries courts, afin d’éviter les erreurs Windows liées au remplacement `*.tmp -> *.json`.

Tables principales :

- `users`
- `contacts`
- `tags`
- `contact_tags`
- `consent_logs`
- `requests`
- `loyalty_accounts`
- `loyalty_transactions`
- `offers`

Le numéro WhatsApp est unique. Si un contact existe déjà, la fiche est mise à jour, les nouvelles sources sont ajoutées dans l’historique, et les nouveaux centres d’intérêt ou programmes sont fusionnés.

Ce stockage JSON est volontairement choisi pour la V1 Windows : il ne nécessite ni Visual Studio Build Tools, ni Python, ni compilation native.

## Fidélité

Règles incluses dans la V1 :

- 1 passage maximum par client et par jour
- passage validé si achat particulier et montant >= 25 000 Ar
- les achats particuliers comptent dans le cumul, même sous 25 000 Ar
- cadeau disponible après 10 passages
- super bonus disponible à chaque palier de 1 500 000 Ar
- excédent reporté sur le cycle suivant
- historique des achats, passages et remises

## Export WhatsApp

La page d’exports permet de choisir le canal marketing :

- WhatsApp
- E-mail
- SMS

L’export CSV et la copie de liste excluent automatiquement :

- les contacts sans consentement actif pour le canal choisi
- les contacts désinscrits
- les contacts avec l’étiquette `Ne plus contacter`
- les contacts sans coordonnée valide pour le canal choisi

Les exports marketing peuvent être filtrés par centre d’intérêt, statut commercial et préférence de créneau. Le CSV inclut aussi les programmes, centres d’intérêt, statuts commerciaux, étiquettes de gestion et coordonnées disponibles.

Dans les formulaires de demande, le canal utilisé pour répondre à la demande est séparé des consentements marketing. Une personne peut demander à être recontactée pour sa réservation sans accepter de recevoir des offres.

WhatsApp reste le canal marketing prioritaire. Le SMS est uniquement utilisé comme solution de secours quand le client n’a pas indiqué de numéro WhatsApp mais a donné un téléphone principal.

Les demandes publiques ajoutent automatiquement le statut commercial `À relancer`. Si la demande contient une date avec un nombre de personnes, ou une précision claire, le statut `Prospect chaud` est aussi ajouté.

## Import Sondage Moments Conviviaux

La page `/admin/imports/sondage-moments-conviviaux` permet d’importer le CSV exporté par l’ancien sondage.

Elle affiche un aperçu avant import, puis consolide les réponses avec les contacts existants quand le même WhatsApp est trouvé. Les réponses importées ajoutent la source `Sondage Moments Conviviaux`, la campagne `moments-conviviaux`, l’intérêt `Moments conviviaux`, la préférence de créneau, et les statuts nécessaires (`À relancer`, `Prospect chaud` ou `À vérifier`).

L’export WhatsApp permet ensuite de filtrer par `Moments conviviaux` et par créneau : `Samedi soir`, `Dimanche après-midi` ou `Les deux`.

## Limites de la V1

- Pas d’intégration WhatsApp API.
- Pas de gestion avancée des rôles côté interface, même si les rôles existent côté API.
- Pas d’import Excel/CSV.
- Pas de suppression définitive des contacts.
- Les corrections/annulations fidélité sont structurées dans l’historique mais restent à enrichir avec une interface dédiée.
- Le stockage JSON convient à une V1 locale et simple, mais une migration PostgreSQL/Supabase sera préférable pour un usage multi-postes.

## Améliorations possibles

- Import CSV de contacts existants.
- Interface dédiée aux corrections fidélité.
- Audit complet des actions admin.
- Connexion Supabase/PostgreSQL.
- Migration vers SQLite via `sql.js` si l’on veut conserver une logique SQL sans compilation native.
- Envoi WhatsApp via API officielle.
- Export Excel natif.
- Statistiques par campagne, canal et période.
=======
# bons_voisins
>>>>>>> 7bd91fb4e03d759fac7a3099b18e722ad0d42878
