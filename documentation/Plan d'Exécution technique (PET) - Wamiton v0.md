# Plan d'Exécution technique (PET) - Wamiton v0

## 0. Resume executif (fonctionnalites et outils)
## 0.1 Fonctionnalites arretees (V0)
| Domaine | Decision V0 |
|---|---|
| Parcours public | Listing evenements + detail + tarifs |
| Compte utilisateur | OTP, sans guest checkout |
| Achat | Panier + commande + paiement Fedapay |
| Billetterie | Emission QR unique apres paiement confirme |
| Distribution | Envoi billet par email + WhatsApp |
| Organisateur | Module scan/check-in avec anti-duplication |
| Plateforme | Web responsive + PWA |

## 0.2 Fonctionnalites a debattre
| Sujet | Option A | Option B | Decision cible |
|---|---|---|---|
| Scan offline | Mode degrade avec synchro differee | Full offline plus complexe | Sprint 3 (POC + arbitrage) |
| Module scanner | Inclus V0 obligatoire | Inclus V0 si risques maitrises | Fin Sprint 3 |
| Push notifications | Decalees en V1 | Ajoutees en V0 | Sprint 4 (go/no-go) |
| 2FA organisateur | Optionnelle | Obligatoire | Sprint 1 |

## 0.3 Outils/techno arretes
| Couche | Outil retenu |
|---|---|
| Frontend | Next.js + PWA |
| Backend metier | Python (FastAPI) |
| Base de donnees | PostgreSQL |
| Admin/CMS | Directus |
| Paiement | Fedapay |
| Jobs async | Redis + worker |
| Notifications | SMTP + WhatsApp API |
| Observabilite | Sentry + logs centralises + uptime |
| Integration annexe | n8n (hors transaction critique) |

## 0.4 Outils/techno a debattre
| Sujet | Option A | Option B | Decision cible |
|---|---|---|---|
| Hosting V0 | PaaS (Render/Railway/Fly) | VPS Docker Compose | Sprint 0 |
| Worker framework Python | Celery | RQ | Sprint 0 |
| Strategie QR token | HMAC | JWT court | Sprint 2 |
| Stockage billet | Generation a la volee | Archive PDF en stockage objet | Sprint 2 |

## 1. Objet
Ce PET sert de base de pilotage technique pour:
- planifier le developpement V0 avec un ordre de build clair,
- cadrer les epics pour sprint planning,
- proposer des titres de tickets (le corps est volontairement a remplir par l equipe).

Source: [README.md](../README.md)

## 2. Attendus fonctionnels arretes (V0)
1. Parcours public: voir les evenements, consulter details et tarifs.
2. Compte utilisateur: creation/connexion avec OTP, sans guest checkout.
3. Parcours achat: panier, commande, paiement Fedapay.
4. Emission billet: QR unique apres paiement confirme.
5. Distribution billet: email + WhatsApp.
6. Historique: retrouver ses billets.
7. Parcours organisateur: module scan/check-in avec anti-duplication.
8. Cible execution: Web responsive + PWA, module scanner priorise en fin de V0.

## 3. Stack technique de reference (V0)
| Couche | Choix V0 | Standard d implementation |
|---|---|---|
| Frontend | Next.js Web + PWA | App Router, TypeScript strict, formulaire valide cote client + serveur |
| Backend metier | Python (FastAPI recommande) | Architecture modulaire, schemas Pydantic, OpenAPI maintenue |
| Base de donnees | PostgreSQL | Migrations versionnees, contraintes DB strictes, index metier |
| Admin/CMS | Directus | CRUD contenu/admin uniquement, pas de logique transactionnelle critique |
| Paiement | Fedapay | Webhook signe, idempotence forte, reconciliation transactionnelle |
| Async jobs | Redis + worker (Celery/RQ) | Retries, DLQ, outbox pattern |
| Automations annexes | n8n | Hors transaction critique (alerts, exports, relances) |
| Notifications | SMTP + WhatsApp API | Templates versionnes, suivi delivrabilite |
| Observabilite | Sentry + logs centralises + uptime | Alertes sur paiements, webhooks, jobs, disponibilite API |
| Infra | Docker Compose local + STAGING/RAT/PROD | CI/CD vers staging, secrets par environnement |

## 4. Bonnes pratiques de mise en place (par techno)
## 4.1 Backend Python (FastAPI)
- Separer `domain`, `application`, `infrastructure`, `api` pour eviter le couplage.
- Encadrer chaque use case critique dans une transaction DB explicite.
- Exposer des endpoints idempotents pour tout ce qui cree des commandes/billets.
- Versionner l API (`/v1`) des le debut.
- Centraliser la gestion des erreurs metier avec codes stables.

## 4.2 PostgreSQL
- Creer les tables critiques en premier: `users`, `events`, `orders`, `payments`, `tickets`, `checkins`.
- Poser des contraintes uniques metier (`provider_tx_id`, `ticket_code`, `order_ref`).
- Ajouter `created_at`, `updated_at`, `deleted_at` sur les tables metier.
- Indexer les requetes frequentes (event date, order status, ticket lookup).
- Interdire les suppressions physiques sur les flux financiers (soft delete + audit).

## 4.3 Directus
- Limiter Directus au contenu/admin: events, medias, categories, moderation.
- Isoler les permissions par roles (`admin`, `editor`, `support`).
- Eviter de brancher Directus sur les tables transactionnelles sensibles.
- Journaliser les actions admin impactantes (publication event, desactivation).

## 4.4 Paiement Fedapay + webhooks
- Verifier signature webhook avant toute ecriture.
- Stocker tous les webhooks recus, meme invalides (audit).
- Rendre le traitement idempotent via cle unique provider.
- Ne jamais emettre un billet sur callback client seul, seulement apres confirmation serveur.
- Mettre en place une tache de reconciliation quotidienne paiement/commande.

## 4.5 QR et check-in
- Signer le QR avec secret backend (HMAC/JWT court).
- Valider le check-in en transaction atomique `ticket not checked_in -> checked_in`.
- Retourner un statut explicite: `valid`, `already_checked_in`, `invalid`.
- Prevoir un journal de scans pour anti-fraude et support terrain.

## 4.6 Notifications
- Utiliser une table `notification_jobs` avec statut et nombre de retries.
- Centraliser templates email/WhatsApp avec version.
- Implementer un fallback manuel en cas d echec de delivrance.

## 4.7 Observabilite et securite
- Instrumenter API avec correlation id.
- Mettre du rate limiting sur auth, checkout et webhook endpoints.
- Activer Sentry backend/frontend des sprint 1.
- Definir runbook incident: paiement valide sans billet emis.

## 5. Ordre recommande de developpement des briques
| Ordre | Brique | Pourquoi maintenant | Gate de sortie |
|---|---|---|---|
| B0 | Foundations (repo, CI, env, secrets, observabilite de base) | Evite dette ops immediate | Staging deployable et monitorable |
| B1 | Modele de donnees V0 + migrations | Tout le reste depend du schema | Migrations appliquees + contraintes en place |
| B2 | Auth + roles + comptes organisateur | Securise les parcours des le debut | Login/OTP stable + ACL minimale |
| B3 | Catalogue events (Directus + API public) | Permet une premiere demo fonctionnelle | Listing + detail + CRUD admin |
| B4 | Orders + paiement Fedapay | Coeur business critique | Commande payee tracee de bout en bout |
| B5 | Emission billet QR + notifications | Livrable metier principal apres paiement | Billet emis et envoye automatiquement |
| B6 | Check-in + anti-fraude | Validation terrain evenement | 1 scan = 1 entree garantie |
| B7 | Hardening prod (backup, alerting, runbook, UAT) | Reduction risque go-live | Checklist GO PROD validee |

## 6. Epics proposes (niveau macro)
| Epic | Objectif | Contenu principal | Hors scope |
|---|---|---|---|
| E1 Foundations & Envs | Installer le socle execution | CI/CD, Docker, secrets, envs, conventions | Optimisations perf avancees |
| E2 Data Platform | Stabiliser le modele metier | Schema V0, migrations, index, audit | BI/warehouse |
| E3 Identity & Access | Controler acces et roles | OTP, sessions, ACL users/orga/admin | SSO enterprise |
| E4 Event Catalog | Operer le catalogue event | Directus, API listing, details, medias | Recommandation IA |
| E5 Checkout & Payments | Fiabiliser monetic | panier, commandes, Fedapay, webhooks | Multi-provider en prod |
| E6 Ticket Issuance | Delivrer billet sans erreur | QR signe, billet, email, WhatsApp | Wallet pass iOS/Android |
| E7 Check-in & Fraud Control | Controler entree terrain | scan, anti-duplication, logs scan | Offline complet sans compromis |
| E8 Reliability & Go-Live | Assurer readiness prod | Sentry, backups, runbooks, UAT | HA multi-region |

## 7. Sprints possibles (2 semaines / sprint)
## Sprint 0 - Cadrage execution + setup technique
Objectif: rendre le socle deployable et poser le modele de donnees V0.
Epics: E1, E2.
Sortie attendue: environnement staging operationnel + migrations initiales.

## Sprint 1 - Auth et catalogue events
Objectif: sortir un produit consultable avec acces comptes.
Epics: E3, E4.
Sortie attendue: parcours consultation event + login OTP + backoffice contenu.

## Sprint 2 - Checkout et paiements
Objectif: fermer le flux commande -> paiement confirme.
Epics: E5.
Sortie attendue: commande payee via webhook idempotent et tracabilite complete.

## Sprint 3 - Billet, notifications et check-in
Objectif: fermer le flux paiement -> billet -> entree evenement.
Epics: E6, E7.
Sortie attendue: billet QR emis, distribue, et check-in anti-dup fonctionnel.

## Sprint 4 - Stabilisation et go-live
Objectif: reduire risque prod et lancer V0.
Epics: E8.
Sortie attendue: UAT validee, runbooks, backups testes, GO PROD.

## 8. Titres de tickets proposes (corps a completer par l equipe)
## 8.1 Sprint 0
- `[E1] Initialiser monorepo web-api-admin avec conventions de code`
- `[E1] Mettre en place Docker Compose local (api, web, postgres, redis, directus)`
- `[E1] Configurer CI lint + tests + build pour web et api`
- `[E1] Configurer deploiement automatique vers STAGING`
- `[E1] Definir standard de secrets et variables par environnement`
- `[E2] Creer migration initiale schema V0 (users/events/orders/payments/tickets/checkins)`
- `[E2] Ajouter contraintes metier uniques sur references paiement et billets`
- `[E2] Ajouter index metier sur recherche events et statuts commandes`

## 8.2 Sprint 1
- `[E3] Implementer inscription et connexion OTP utilisateur`
- `[E3] Implementer roles et permissions utilisateur/organisateur/admin`
- `[E3] Ajouter endpoint profil compte et historique de sessions`
- `[E4] Configurer collections Directus pour events, categories et medias`
- `[E4] Implementer API listing events avec filtres principaux`
- `[E4] Implementer page detail evenement et affichage ticket types`
- `[E4] Ajouter moderation admin publication/depublication event`

## 8.3 Sprint 2
- `[E5] Implementer panier et creation de commande`
- `[E5] Implementer state machine commande/paiement`
- `[E5] Integrer API Fedapay pour initier paiement`
- `[E5] Implementer endpoint webhook Fedapay avec verification signature`
- `[E5] Rendre le traitement webhook idempotent par cle transaction provider`
- `[E5] Ajouter reconciliation quotidienne commandes vs transactions`
- `[E5] Ajouter ecran support pour relancer une commande en echec`

## 8.4 Sprint 3
- `[E6] Implementer generation QR signe cote backend`
- `[E6] Implementer emission billet apres paiement confirme`
- `[E6] Implementer envoi billet par email (SMTP) avec retries`
- `[E6] Implementer envoi billet par WhatsApp API avec retries`
- `[E6] Ajouter historique billets dans espace utilisateur`
- `[E7] Implementer endpoint check-in atomique anti-duplication`
- `[E7] Implementer interface scanner organisateur (mode online V0)`
- `[E7] Ajouter journal des scans avec statuts de validation`
- `[E7] Prototyper strategie offline scan et rapport de limites`

## 8.5 Sprint 4
- `[E8] Integrer Sentry frontend et backend avec alertes critiques`
- `[E8] Centraliser logs applicatifs avec correlation id`
- `[E8] Mettre en place backups automatiques PostgreSQL`
- `[E8] Realiser test restore et documenter procedure`
- `[E8] Rediger runbook incident paiement valide sans billet`
- `[E8] Executer campagne UAT complete et suivre anomalies`
- `[E8] Construire checklist GO/NO-GO et valider lancement prod`

## 9. Regles de priorisation pour sprint planning
- Priorite P0: tout ce qui bloque paiement, emission billet, check-in.
- Priorite P1: tout ce qui bloque observabilite, securite, support operationnel.
- Priorite P2: confort utilisateur et optimisations non critiques V0.
- Aucune story fonctionnelle ne passe en done sans trace technique exploitable.

## 10. Definition of Done (projet)
Un ticket est considere `Done` si:
- code merge et relu,
- tests prevus executes et verts,
- logs et erreurs exploitables en environnement cible,
- documentation mise a jour dans `documentation/`,
- verification fonctionnelle realisee sur l environnement du sprint.
