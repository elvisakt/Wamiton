# Document d'Architecture Technique (DAT) - Wamiton v0

## 1. Metadonnees document
| Champ | Valeur |
|---|---|
| Projet | Wamiton |
| Version | 1.0 |
| Statut | Draft de reference V0 |
| Date | 23/04/2026 |
| Base documentaire | `README.md` + `Plan d'Exécution technique (PET) - Wamiton v0.md` |
| Public cible | Tech lead, dev backend/frontend, ops, produit |

## 2. Objectif et perimetre
## 2.1 Objectif du DAT
Definir l architecture technique cible V0 pour garantir:
- un flux achat billet fiable de bout en bout,
- un check-in robuste anti-duplication,
- une mise en production exploitable et monitorable.

## 2.2 Perimetre V0
- Application web responsive + PWA.
- Authentification OTP utilisateur, sans guest checkout.
- Catalogue events + details + tarifs.
- Panier, commande, paiement Fedapay.
- Emission billet QR apres confirmation serveur paiement.
- Envoi billet par email et WhatsApp.
- Check-in organisateur avec anti-fraude.
- Environnements DEV/STAGING/RAT/PROD.

## 2.3 Hors perimetre V0
- Application mobile grand public dediee.
- Wallet pass iOS/Android.
- Multi-provider paiement en production.
- IA/recommandation avancee.
- Architecture HA multi-region.

## 3. Exigences de reference
## 3.1 Exigences fonctionnelles (EF)
| ID | Exigence | Critere d acceptance |
|---|---|---|
| EF-01 | Consulter liste et details des evenements | Listing filtre + detail event disponibles |
| EF-02 | Creer un compte et se connecter via OTP | OTP valide, session ouverte, role applique |
| EF-03 | Passer une commande et payer | Commande tracee avec statut coherent |
| EF-04 | Emettre un billet apres paiement confirme | Billet unique genere apres webhook valide |
| EF-05 | Envoyer le billet au client | Email/WhatsApp envoyes avec suivi statut |
| EF-06 | Permettre check-in organisateur | Validation ticket en moins de 1 action |
| EF-07 | Empêcher double entree | Ticket deja scanne rejete explicitement |
| EF-08 | Afficher historique billets utilisateur | Liste des billets accessibles en compte |
| EF-09 | Administrer le catalogue | CRUD events via backoffice Directus |

## 3.2 Exigences non fonctionnelles (ENF)
| ID | Exigence | Cible V0 |
|---|---|---|
| ENF-01 | Disponibilite | >= 99.5% sur PROD |
| ENF-02 | Performance API lecture | p95 < 500 ms |
| ENF-03 | Performance check-in | p95 < 300 ms |
| ENF-04 | Integrite transactionnelle | 0 double emission billet pour une transaction provider |
| ENF-05 | Securite paiement/webhook | Signature validee + idempotence obligatoire |
| ENF-06 | Traçabilite | Logs corrélés par `trace_id` |
| ENF-07 | Exploitabilite | Alerting actif sur paiement, webhook, jobs, uptime |
| ENF-08 | Continuite | Backups quotidiens, RPO 24h, RTO 4h |
| ENF-09 | Maintainabilite | Migrations versionnees + architecture modulaire |

## 4. Decisions d architecture
| ID | Decision | Statut | Justification |
|---|---|---|---|
| ADR-01 | Frontend web + PWA | Retenue | Time-to-market et diffusion rapide |
| ADR-02 | Backend metier Python FastAPI modulaire | Retenue | Vitesse de dev + API typed + docs OpenAPI |
| ADR-03 | PostgreSQL comme source de verite | Retenue | Integrite transactionnelle et contraintes fortes |
| ADR-04 | Directus limite au CMS/admin | Retenue | Acceleration CRUD sans exposer la logique critique |
| ADR-05 | Paiement Fedapay via webhook serveur | Retenue | Fiabilite et securite du flux monétique |
| ADR-06 | Redis + worker pour taches async | Retenue | Resilience envoi notifications et traitements differts |
| ADR-07 | Sentry + logs centralises + uptime | Retenue | Detection rapide des incidents |
| ADR-08 | Hebergement V0 PaaS vs VPS | A arbitrer | Compromis vitesse de mise en place vs controle cout |
| ADR-09 | Worker Python Celery vs RQ | A arbitrer | Complexite/robustesse du traitement async |
| ADR-10 | Scan offline (degre V0) | A arbitrer | Risque produit terrain vs complexite technique |

## 5. Architecture cible V0
## 5.1 Vue logique des composants
| Composant | Responsabilites | Donnees manipulees |
|---|---|---|
| Frontend Next.js/PWA | UI publique, panier, espace compte, espace organisateur scan | Sessions, vues catalogue, etat panier |
| API FastAPI | Use cases metier, orchestration commandes/paiements/billets/check-in | Orders, payments, tickets, checkins |
| PostgreSQL | Persistance transactionnelle et audit | Tables metier et historiques |
| Directus | Administration contenu et moderation | Events, medias, categories |
| Worker async | Envoi emails/WhatsApp, retries, jobs metier non bloquants | notification_jobs, outbox |
| Integrations externes | Fedapay, SMTP, WhatsApp | Webhooks, messages, status provider |
| Observabilite | Sentry, logs, uptime monitor | erreurs, metrics, traces |

## 5.2 Flux critique F1 - Achat vers billet
1. Utilisateur cree/ouvre session.
2. Selection ticket type et creation commande `order_draft`.
3. Appel Fedapay pour initier paiement `pending`.
4. Reception webhook Fedapay cote serveur.
5. Verification signature + validation idempotence.
6. Ecriture transactionnelle `payment=paid`, `order=paid`.
7. Generation ticket + QR signe.
8. Enqueue notifications email/WhatsApp.
9. Mise a disposition billet dans historique utilisateur.

Regle cle: aucune emission billet ne depend d un callback front uniquement.

## 5.3 Flux critique F2 - Check-in
1. Organisateur scanne QR.
2. API verifie signature QR et validite ticket.
3. Transaction atomique `not_checked_in -> checked_in`.
4. Retour statut `valid` ou `already_checked_in` ou `invalid`.
5. Journalisation du scan (agent, terminal, timestamp, resultat).

## 5.4 Modele de donnees macro
| Entite | Cle primaire | Contraintes cles |
|---|---|---|
| users | `id` | unique email/phone, role obligatoire |
| events | `id` | date_event indexee, statut publication |
| ticket_types | `id` | FK event, quota/capacite |
| orders | `id` | unique `order_ref`, status indexe |
| payment_transactions | `id` | unique `provider_tx_id`, FK order |
| webhook_events | `id` | unique `provider_event_id`, statut traitement |
| tickets | `id` | unique `ticket_ref`, FK order+user, hash QR unique |
| checkins | `id` | unique `(ticket_id)` pour anti-duplication |
| notification_jobs | `id` | statut, retries, canal, correlation_id |

## 5.5 Regles d integrite et idempotence
- `payment_transactions.provider_tx_id` unique.
- `webhook_events.provider_event_id` unique.
- `checkins.ticket_id` unique.
- Toute transition `order_status` passe par une state machine explicite.
- Toute operation critique est transactionnelle.

## 6. Interfaces et integrations
## 6.1 Fedapay
- Usage: creation paiement + webhook confirmation.
- Controles: signature, horodatage, anti-replay, idempotence.
- Journalisation: payload brut + statut traitement.

## 6.2 SMTP
- Usage: envoi billet et notifications transactionnelles.
- Bonnes pratiques: templates versionnes, retries, bounce monitoring.

## 6.3 WhatsApp API
- Usage: envoi billet et confirmation achat.
- Bonnes pratiques: fallback email en cas d echec canal WhatsApp.

## 6.4 Directus
- Usage: contenu catalogues, medias, moderation.
- Contrainte: aucune ecriture transactionnelle paiement/check-in via Directus.

## 6.5 n8n
- Usage: relances, exports, alertes non critiques.
- Contrainte: pas de logique transactionnelle bloqueante.

## 7. Securite et conformite technique
## 7.1 Authentification et autorisation
- OTP avec expiration courte et limitation des tentatives.
- RBAC minimal: `user`, `organisateur`, `admin`.
- Controle d acces serveur sur endpoints sensibles.

## 7.2 Protections API
- Rate limiting sur auth, checkout, webhook.
- Validation stricte des payloads entres.
- CORS strict par environnement.

## 7.3 Secrets et donnees sensibles
- Secrets uniquement par variables d environnement.
- Rotation des cles paiement et webhook.
- Aucune cle en dur dans le code.

## 7.4 Journalisation et audit
- Logs structurels JSON avec `trace_id`, `user_id`, `order_ref`.
- Audit trail pour actions admin et scans check-in.
- Conservation des traces webhook pour forensic.

## 8. Environnements, deploiement et exploitation
## 8.1 Environnements
| Environnement | Usage | Donnees |
|---|---|---|
| DEV | Developpement local | Donnees de test |
| STAGING | Integration continue | Sandbox providers |
| RAT | Recette metier/UAT | Donnees anonymisees |
| PROD | Exploitation reelle | Donnees reelles |

## 8.2 Pipeline CI/CD minimal
1. Lint + tests unitaires.
2. Build images.
3. Tests integration (API + DB + webhook simulé).
4. Deploy auto STAGING.
5. Promotion manuelle RAT puis PROD apres GO.

## 8.3 Supervision et alerting
- Sentry frontend/backend actif.
- Uptime checks sur API et frontend.
- Alertes P1: webhook en echec, job queue bloquee, taux echec paiement anormal.
- Dashboard ops: commandes par statut, emissions billet, scans check-in.

## 8.4 Sauvegarde et restauration
- Backup PostgreSQL quotidien automatique.
- Retention minimale 30 jours.
- Test restore mensuel obligatoire documente.
- Runbook incident versionne dans `documentation/`.

## 9. Strategie qualite
## 9.1 Niveaux de test
- Unitaires: logique metier state machine, generation QR, transitions statut.
- Integration: paiement/webhook, emission billet, check-in atomique.
- End-to-end: parcours complet utilisateur et organisateur.
- UAT: scenarios metier en RAT avant go-live.

## 9.2 Quality gates
- Build vert obligatoire avant merge.
- Tests critiques paiement/check-in obligatoires.
- Aucune release PROD sans validation UAT + checklist GO/NO-GO.

## 10. Risques majeurs et plans de mitigation
| Risque | Impact | Mitigation |
|---|---|---|
| Mauvaise gestion webhook/idempotence | Double billet, incident financier | Contraintes DB + tests integration + replay tests |
| Complexite scan offline | Retard sprint, dette technique | POC borne en Sprint 3 + decision explicite |
| Observabilite insuffisante | Detection tardive incidents | Sentry+alerting des Sprint 1-2 |
| Derive scope V0 | Delai go-live non tenu | Gouvernance stricte scope/hors scope |

## 11. Gouvernance technique
## 11.1 Roles minimaux
- Tech lead: valide decisions architecture et arbitrages.
- Backend lead: garantit integrite transactionnelle et contrats API.
- Frontend lead: garantit parcours UX et robustesse UI.
- Ops/DevOps: garantit pipeline, monitoring, backups.
- Product owner: valide priorites et acceptation metier.

## 11.2 Rituel de pilotage recommande
- Revue architecture hebdomadaire (30 min).
- Revue risques hebdomadaire.
- Revue readiness avant chaque promotion d environnement.

## 12. Traceabilite avec le PET
Le PET contient le plan d execution (epics, sprints, titres de tickets).  
Le DAT fixe le cadre d architecture, d exigences et d exploitation sur lequel ce plan doit s executer.

| Epic PET | Sections DAT de reference |
|---|---|
| E1 Foundations & Envs | 8, 11 |
| E2 Data Platform | 5.4, 5.5 |
| E3 Identity & Access | 7.1, 7.2 |
| E4 Event Catalog | 5.1, 6.4 |
| E5 Checkout & Payments | 5.2, 6.1 |
| E6 Ticket Issuance | 5.2, 6.2, 6.3 |
| E7 Check-in & Fraud Control | 5.3, 7.4 |
| E8 Reliability & Go-Live | 8, 9, 10 |

## 13. Points ouverts a clore avant GO PROD
1. Choix hebergement final V0 (PaaS ou VPS).
2. Choix worker async (Celery ou RQ).
3. Niveau d offline scanner acceptable en V0.
4. Politique retention logs et conformite locale.

