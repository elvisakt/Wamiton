# Wamiton — Cadrage technique (2h)
*(Support d’atelier 2 × 1h — orienté décisions techniques, infra, environnements, user stories)*

## Rappel — C’est quoi un cadrage technique ?
Un **cadrage technique** est une étape où **nous prenons les décisions techniques structurantes** avant d’industrialiser le dev : architecture, stack, environnements, infra, sécurité, delivery, outils, et critères techniques de réussite (fiabilité, coûts, performance, maintenabilité).  
L’objectif est de **réduire le risque** (paiement, billets, fraude, prod) et d’accélérer la livraison.

---

# Organisation (2 sessions de 1h)

## Session 1 (1h, ce soir) — “Décisions bloquantes + socle”
Objectif : sortir avec **des décisions claires** sur la plateforme (web/mobile), la forme du backend, la stratégie infra initiale, et la structure des environnements DEV/STAGING/RAT/PROD.

### Plan minute par minute (1h)
- 0:00–0:05 — Cadre & livrables de la session
- 0:05–0:20 — Plateforme : Web/PWA vs Mobile (dès V0 ?) + app scanner
- 0:20–0:40 — Backend & données : architecture, modules, Directus/n8n/Python
- 0:40–0:55 — Infra V0 : Docker/Compose vs PaaS vs K8s + environnements
- 0:55–1:00 — Décisions, questions ouvertes, actions avant session 2

**Livrables session 1**
- Decision log (choix validés + owner)
- Architecture cible “V0” (briques + responsabilités)
- Stratégie environnements + déploiement de base

---

## Session 2 (1h, plus tard) — “Prod readiness + challenge user stories”
Objectif : verrouiller la chaîne paiement/billets (webhooks, idempotence, anti-fraude), établir le plan observabilité/sauvegardes, et challenger le backlog avec des critères techniques + stories manquantes.

### Plan minute par minute (1h)
- 0:00–0:10 — Revue des décisions session 1 + ce qui a changé
- 0:10–0:35 — Paiement & billets : flux, statuts, sécurité, contrôle d’accès
- 0:35–0:50 — Ops : monitoring, logs, backups, runbook, coûts
- 0:50–0:58 — Challenge user stories (ajouts techniques + critères d’acceptation)
- 0:58–1:00 — Plan d’exécution (ordre des chantiers, qui fait quoi)

**Livrables session 2**
- Spéc technique paiement/billet/check-in (niveau “implémentable”)
- Liste “Tech user stories” à ajouter au backlog
- Plan “GO PROD” + checklist

---

# Session 1 (1h, ce soir) — contenu détaillé

## 1) Cadre & objectifs (0:00–0:05)
### Questions d’ouverture
- Quelles sont les 3 décisions qui, si on ne les prend pas, bloquent le dev cette semaine ?
- Qu’est-ce qu’on veut absolument avoir tranché à la fin de cette heure ?

### Recommandation
- On sort avec des décisions “suffisamment bonnes” et un plan d’action, plutôt qu’un débat parfait.

---

## 2) Plateforme : commencer par une app mobile ou pas ? (0:05–0:20)

### Points à challenger techniquement
- **Besoin natif immédiat ?**
  - caméra pour scan QR
  - offline à l’entrée
  - notifications push
  - wallet / pass
- **Acquisition & vitesse** : web/PWA = plus rapide à livrer et à diffuser.
- **Risque** : 2 apps (iOS/Android) + web = charge forte au début.

### Questions techniques
- Est-ce que le scan QR doit marcher **même sans réseau** ?
- Est-ce qu’on veut une app séparée “Scanner/Organisateur” ?
- Notre V0 a-t-elle besoin de push notifications ou on peut démarrer par email/WhatsApp ?

### Recommandation (proposée)
- **V0/V1 : Web responsive + PWA**
- **V1.5 : mini app “Scanner” dédiée organisateurs** (React Native Expo) si le scan web montre des limites.
- On garde la possibilité “Mobile grand public” plus tard, quand on a validé usage & acquisition.

### Décision à prendre ce soir
- [ ] V0 = Web/PWA uniquement
- [ ] V0 = Web + app Scanner
- [ ] V0 = Web + mobile grand public (rarement optimal)

---

## 3) Backend & données : comment on structure le cœur du produit ? (0:20–0:40)

### Ce qu’on sait déjà (du backlog)
- Nous visons un MVP avec : liste events, panier, paiement (webhooks), QR, email, compte OTP, historique billets.
- Les flux paiement/billet sont **critiques** : ils imposent un backend métier.

### Modules backend (monolithe modulaire recommandé)
- Auth & Users
- Events
- Ticketing (ticket types, capacité, règles)
- Orders & Payments (state machine)
- Ticket Issuance (QR, email)
- Check-in (validation QR, anti-duplication)
- Notifications
- Admin/Moderation

### Questions techniques (challenge)
- Quel niveau de logique métier est critique dès V0 ?
  - capacité / quotas
  - annulation / remboursement
  - transfert de billet
- Est-ce qu’on autorise l’achat sans compte (guest checkout) ?
- Notre modèle de données minimal ressemble à quoi ?
  - Event, Venue, Category, TicketType, Order, PaymentTransaction, Ticket, CheckIn

### Directus / n8n / Python — comment on les utilise (sans se piéger)
#### Directus
**Utile** pour : admin rapide, CRUD events, médias, rôles, modération.
**Limite** : paiement/billet/check-in -> backend métier obligatoire.

✅ Recommandation  
- On utilise **Directus comme CMS/admin** branché à PostgreSQL pour gagner du temps (création events, modération).
- On garde un **backend applicatif** pour le transactionnel.

#### n8n
**Utile** pour : automations périphériques (emails, alerting, exports, sync).
**Limite** : ne pas mettre la logique transactionnelle critique dedans.

✅ Recommandation  
- n8n en “périphérie” uniquement, optionnel.

#### Python
**Utile** pour : reco/IA, data pipelines, analytics.
**Limite** : si on n’a pas besoin tout de suite, ça peut ralentir la V0.

✅ Recommandation  
- Backend principal en **TypeScript** (si on est déjà sur Next.js).
- On garde Python pour **une brique reco** plus tard si nécessaire.

### Décisions à prendre ce soir
- [ ] Backend TS : NestJS (recommandé) / Express modulaire
- [ ] DB : PostgreSQL
- [ ] Admin : Directus oui/non
- [ ] Jobs async : Redis + worker oui/non (recommandé)

---

## 4) Infra V0 & environnements : Docker, PaaS, ou Kubernetes ? (0:40–0:55)

### Objectif
Mettre en place un setup **simple, stable et répétable** pour DEV + STAGING, et préparer RAT/PROD.

### Options infra
1) **PaaS** (Railway/Render/Fly)  
   - + rapide, ops simple  
   - - verrouillage, coûts à surveiller
2) **VPS + Docker Compose**  
   - + contrôle, coût prévisible  
   - - plus d’ops (mises à jour, backups)
3) **Kubernetes**  
   - + scaling/HA  
   - - complexité forte (pas adapté V0 si on n’a pas déjà l’expertise)

✅ Recommandation  
- **V0/MVP : PaaS ou VPS + Docker Compose**
- **Pas de Kubernetes** en V0 (on y revient quand on a de la charge et des besoins HA/scaling).

### Environnements (définitions)
- DEV : local (compose), sandbox paiement
- STAGING : intégration (auto deploy), sandbox paiement
- RAT : recette “comme prod”, validation fonctionnelle avant prod
- PROD : vraies clés paiement, monitoring + backups

### Questions techniques
- Qui déploie ? (CI/CD automatisé ou manuel)
- Quels domaines ? (`api-staging`, `api-rat`, `api`)
- Comment on gère secrets & clés (env vars) ?
- Où sont les backups Postgres ? (automatiques + test restore)

✅ Recommandation (ordre)
- Sprint 1 : DEV + STAGING
- Sprint 3 : ajouter RAT (avant vraie mise en prod paiement)
- PROD : uniquement quand checklist “GO PROD” est prête

### Décisions à prendre ce soir
- [ ] PaaS (Railway/Render/Fly) OU VPS Docker Compose
- [ ] Mise en place STAGING dès Sprint 1
- [ ] RAT activé avant brancher paiement prod

---

## 5) Wrap-up & actions avant session 2 (0:55–1:00)
- On remplit le Decision Log
- On liste 3 “open questions” maximum
- On assigne un owner pour chaque action (ex : tester provider SMS/Email, lister contraintes scan offline, etc.)

---

# Session 2 (1h, plus tard) — contenu détaillé

## 1) Paiement & billets : flux robuste (0:10–0:35)

### Questions techniques incontournables
- Webhooks : comment on gère les doublons ? (**idempotence**)
- Statuts : quelle state machine ?
  - order_draft → pending → paid → ticket_issued → checked_in
  - canceled/refunded si nécessaire
- Signature webhook & sécurité (replay protection)
- Génération billet :
  - QR contient quoi ? (token signé + identifiant)
  - stockage du billet (PDF/image) ou génération “à la volée”
- Anti-fraude :
  - 1 scan = 1 entrée (check-in atomique en DB)
  - offline : comment on synchronise ?

✅ Recommandation
- State machine claire + table `PaymentTransaction`
- Webhook idempotent (clé unique = provider_tx_id)
- QR = token signé (HMAC/JWT court) + vérification serveur
- Check-in = endpoint qui fait un update transactionnel (atomic)

---

## 2) Ops & prod readiness (0:35–0:50)

### Questions techniques
- Monitoring : comment on sait que ça marche ?
- Logs : où vont-ils ? (centralisation)
- Erreurs : comment on les voit ? (Sentry)
- Backups : fréquence, rétention, test restore
- Runbook : quoi faire si paiement ok mais ticket non émis ?

✅ Recommandation (minimum viable prod)
- Sentry + Uptime monitor
- Backups DB automatiques + test restore mensuel
- Alerting sur :
  - erreurs webhooks
  - taux d’échec paiement
  - file de jobs bloquée
- Procédures manuelles de correction (réémettre un billet, etc.)

---

## 3) Challenge user stories (0:50–0:58)

### Méthode
Pour chaque user story critique, on ajoute :
- **dépendances techniques**
- **risques**
- **critères d’acceptation techniques**
- **observabilité** (comment on mesure que ça marche)

### Exemples de stories techniques à ajouter
- Modèle de données V0 (migrations)
- Upload médias & stockage
- Worker jobs + retry
- Rate limiting & anti-spam
- Endpoint check-in + anti-duplication
- Idempotence webhooks
- Template emails + deliverability
- Secrets management (staging/rat/prod)

✅ Recommandation
- On ajoute ces stories au backlog, sinon on va les faire “en urgence” au pire moment.

---

# Recommandations finales (résumé)
- Plateforme V0 : **Web responsive + PWA** (app scanner plus tard si besoin)
- Backend : **TypeScript** (NestJS recommandé) + PostgreSQL
- Admin : **Directus** (gain rapide) + backend métier pour transactionnel
- Jobs : **Redis + worker**
- Infra V0 : **PaaS ou VPS + Docker Compose**
- Environnements : DEV + STAGING (Sprint 1), RAT (Sprint 3), PROD après checklist GO PROD

---

# Decision Log (à remplir)
| Sujet | Options | Décision | Owner | Date |
|---|---|---|---|---|
| Plateforme V0 | Web/PWA vs Web+Mobile |  |  |  |
| App scanner | Web scan vs app dédiée |  |  |  |
| Backend | NestJS vs Express / Python |  |  |  |
| Admin | Directus oui/non |  |  |  |
| Jobs async | Redis+worker oui/non |  |  |  |
| Infra | PaaS vs VPS Docker |  |  |  |
| Environnements | DEV/STAGING/RAT/PROD |  |  |  |
| Observabilité | Sentry/Uptime/logs |  |  |  |
| Backups | stratégie + test restore |  |  |  |
| Paiement | provider(s) + webhooks |  |  |  |
