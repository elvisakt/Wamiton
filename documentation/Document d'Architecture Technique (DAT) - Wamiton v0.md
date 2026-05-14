# Document d'Architecture Technique (DAT) - Wamiton v0

## 1. Objectif
Ce DAT synthetise l'architecture technique cible de Wamiton v0 sur la base de `wamiton_user_stories.md`.

Il sert a:
- aligner l'equipe sur les choix techniques structurants,
- identifier les flux critiques a securiser,
- cadrer les points ouverts avant sprint planning.

## 2. Surfaces applicatives
| Surface | Domaine | Usage | Cible |
|---|---|---|---|
| A - Acheteur | `wamiton.bj` | Decouverte, achat, billets, historique | PWA mobile-first |
| B - Organisateur | `dashboard.wamiton.bj` | Events, ventes, reversements, agents | Web desktop |
| C - Check-in | `checkin.wamiton.bj` | Scan QR, sync offline, entree terrain | PWA mobile ultra-simple |

## 3. Perimetre MVP
## 3.1 Must Have
- Catalogue events: liste, recherche, filtres categorie/ville, detail event.
- Comptes: acheteur, organisateur, agents de controle.
- Achat: panier, quotas, commande, paiement MTN MoMo/Moov via agregateur.
- Billetterie: confirmation commande, emission differee, QR unique, PDF.
- Billets: historique, affichage offline, cession gratuite avant emission.
- Organisateur: creation event, categories de billets, parametres de billetterie.
- Operations event: annulation, remboursement, frais de service, reversements.
- Check-in: scan QR, anti-duplication, synchronisation offline.

## 3.2 Hors scope MVP
- Application native iOS/Android.
- Wallet Apple/Google.
- Revente payante de billets.
- Programme de fidelite.
- Multi-pays UEMOA.
- IA/recommandation avancee.

## 4. Decisions techniques
| Sujet | Decision v0 | Statut |
|---|---|---|
| Backend | Backend Django unique | A confirmer officiellement |
| Base de donnees | PostgreSQL | Retenu |
| Frontends | 3 surfaces web/PWA | Retenu |
| Paiement | Agregateur couvrant MTN MoMo, Moov, refunds, payouts | A choisir |
| Jobs async | Worker pour emission, PDF, notifications, remboursements | Retenu |
| Stockage fichiers | Stockage objet pour images, logos, PDF billets | Recommande |
| Observabilite | Sentry + logs structures + uptime | Retenu |
| Directus | Backoffice interne optionnel, pas dashboard organisateur principal | A arbitrer |

## 5. Architecture cible
| Composant | Responsabilite |
|---|---|
| PWA Acheteur | Catalogue, panier, paiement, billets, cache offline |
| Dashboard Organisateur | Events, billets, ventes, reversements, agents |
| PWA Check-in | Scan QR, liste synchronisee, scans offline |
| Backend Django | API, auth, logique metier, webhooks, state machines |
| PostgreSQL | Source de verite transactionnelle |
| Worker async | Emission differee, QR/PDF, notifications, refunds, payouts |
| Agregateur paiement | Paiements, remboursements, reversements |
| Stockage objet | Medias events, logos, PDF billets |
| Monitoring | Erreurs, logs, disponibilite, alertes |

## 6. Flux critiques
## 6.1 Achat et paiement
1. Acheteur connecte selectionne billets et quantites.
2. Backend verifie quotas, dates de vente et disponibilite.
3. Commande creee en `pending_payment`.
4. Paiement initie via agregateur.
5. Webhook provider valide le paiement.
6. Backend applique idempotence et passe la commande a `paid`.
7. Billets crees en `pending_issuance`.

Regle: aucun billet n'est emis sur simple retour front.

## 6.2 Emission differee
1. Organisateur configure fermeture vente, date emission, deadline cession.
2. Worker detecte les billets a emettre.
3. Backend genere QR unique + PDF.
4. Billet passe a `issued`.
5. Email, WhatsApp et notification in-app sont envoyes.

## 6.3 Cession de billet
1. Possible uniquement avant emission et avant deadline.
2. Cessionnaire obligatoire avec compte actif.
3. Changement de detenteur en transaction.
4. Audit complet de la cession.
5. QR/PDF emis au nom du detenteur final.

## 6.4 Check-in online/offline
1. Agent synchronise la liste billets apres fermeture de vente.
2. Online: validation serveur atomique.
3. Offline: validation locale depuis liste synchronisee.
4. Reconnexion: push des scans offline au serveur.
5. Conflits: premier scan horodate accepte, autres rejetes et audites.

## 6.5 Annulation et remboursement
1. Event passe a `cancelled` avec motif.
2. Acheteurs notifies.
3. Remboursements lances via agregateur.
4. Statuts commandes/paiements/remboursements reconciles.

## 7. Donnees macro
| Domaine | Entites principales |
|---|---|
| Identite | `users`, `organizations`, `organizer_members` |
| Catalogue | `events`, `event_media`, `ticket_types`, `ticketing_settings` |
| Achat | `orders`, `order_items`, `payment_transactions`, `webhook_events` |
| Billets | `tickets`, `ticket_transfers`, `ticket_artifacts` |
| Check-in | `checkins`, `checkin_sync_batches` |
| Finance | `service_fees`, `refund_transactions`, `payouts` |
| Notifications | `notification_jobs` |

## 8. Regles techniques non negociables
- `provider_tx_id`, `provider_event_id`, `order_ref`, `ticket_ref` doivent etre uniques.
- Les paiements, emissions, cessions, refunds et check-ins doivent etre idempotents.
- Les quotas billets doivent etre proteges par transaction DB.
- Un billet final ne peut avoir qu'un seul check-in valide cote serveur.
- Les donnees carte ne sont jamais stockees par Wamiton.
- Les exports acheteurs doivent etre limites et journalises.
- Les logs doivent inclure `trace_id`, `user_id`, `order_ref`, `event_id` quand disponible.

## 9. State machines minimales
| Objet | Etats |
|---|---|
| Commande | `draft -> pending_payment -> paid -> refunded/cancelled` |
| Paiement | `initiated -> pending_provider -> succeeded/failed/expired` |
| Billet | `pending_issuance -> transferred -> issued -> used/cancelled/refunded` |
| Event | `draft -> pending_review -> published -> sales_closed -> completed/cancelled` |
| Reversement | `pending -> approved -> sent -> paid/failed` |

## 10. Environnements
| Env | Usage |
|---|---|
| DEV | Developpement local |
| STAGING | Integration continue + sandbox providers |
| PRE-PROD/RAT | Recette metier |
| PROD | Exploitation reelle |

## 11. Qualite et exploitation
- Tests unitaires: state machines, quotas, cession, emission.
- Tests integration: webhooks, refunds, payouts, check-in offline sync.
- Tests E2E: achat complet, emission billet, PDF, cession, scan.
- Monitoring: Sentry, uptime checks, logs structures.
- Alertes critiques: webhook echec, job emission bloque, taux echec paiement, sync check-in en erreur.
- Backups PostgreSQL quotidiens avec test restore mensuel.

## 12. Risques principaux
| Risque | Mitigation |
|---|---|
| Django vs FastAPI non tranche | Arbitrer avant Sprint 0 et aligner PET/US |
| Offline check-in multi-agents | Sync batch + resolution conflit + audit |
| Emission differee ratee | Jobs supervises + alertes billets a emettre |
| Double emission ou double entree | Idempotence + contraintes DB |
| Donnees personnelles dans exports | Minimisation + permissions + journalisation |
| Remboursements/reversements incomplets | Reconciliation provider quotidienne |

## 13. Points ouverts
1. Confirmer backend Django unique ou reviser les user stories.
2. Clarifier auth cible: email/mot de passe, telephone/mot de passe, OTP.
3. Choisir l'agregateur paiement compatible MTN, Moov, carte, refunds, payouts.
4. Definir le role exact de Directus.
5. Decider stockage PDF: archive systematique ou generation a la demande.
6. Definir les donnees autorisees dans les exports organisateur.
7. Formaliser la strategie de conflit offline check-in.
