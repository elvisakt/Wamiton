# wamiton_user_stories

---

# Table des matières

1. Epic 1 : Découverte et recherche d’événements
2. Epic 2 : Compte et profil acheteur
3. Epic 3 : Achat de billets
4. Epic 4 : Gestion des billets achetés
5. Epic 5 : Compte et profil organisateur
6. Epic 6 : Gestion des événements
7. Epic 7 : Vente et billetterie en ligne
8. Epic 8 : Controle des entrees (Check-in)
9. Epic 9 : Tableau de bord organisateur
10. Recapitulatif par priorité

**Dernière mise a jour** : Ajout du systeme d’emission differee, de cession de billets et des parametres de billetterie organisateur.

---

# Introduction

Ce document liste les fonctionnalites de Wamiton sous forme de User Stories, du point de vue des deux types d’utilisateurs finaux : l’acheteur de billets et l’organisateur d’événements.

**Perimetre** : Plateforme PWA mobile-first de billetterie culturelle au Benin.

**Langues cibles** : Francais (interface principale).

---

# Surfaces applicatives

Le produit Wamiton est compose de trois surfaces distinctes, toutes alimentees par un backend Django unique.

**Surface A — Application acheteur** ([wamiton.bj](http://wamiton.bj))
PWA mobile-first. Permet la decouverte des événements, l’achat de billets et la consultation des billets numeriques. C’est le produit grand public.

**Surface B — Tableau de bord organisateur** ([dashboard.wamiton.bj](http://dashboard.wamiton.bj))
Interface web, optimisee desktop. Permet la creation et la gestion des événements, le suivi des ventes et la gestion des reversements. Utilisee en amont de l’événement, dans un contexte bureautique.

**Surface C — Application check-in** (checkin.wamiton.bj)
PWA mobile ultra-epuree. Permet uniquement le scan des QR codes et la validation des entrees. Utilisee le jour J, sur le terrain, par les agents de controle. Doit fonctionner avec une connexion degradee.

---

# Format et conventions

### Template de User Story

**En tant que** [type d’utilisateur]
**Je veux** [action/fonctionnalite]
**Afin de** [benefice/valeur]

### Priorites MoSCoW

- **Must Have** : Indispensable pour le MVP
- **Should Have** : Important mais pas bloquant
- **Could Have** : Souhaitable si temps disponible
- **Won’t Have** : Hors perimetre pour cette version

---

# Epic 1 : Découverte et recherche d’événements

> Surface : A — Application acheteur
> 

### US-001 : Consulter la liste des événements à venir

**En tant qu’** acheteur de billets
**Je veux** voir une liste des événements culturels à venir au Benin
**Afin de** decouvrir ce qui se passe pres de chez moi sans avoir a chercher sur plusieurs reseaux sociaux

**Critères d’acceptation :**
- [ ] La liste affiche les événements futurs par ordre chronologique par defaut
- [ ] Chaque événement affiche : nom, date, lieu, ville, prix minimal et une image
- [ ] La liste est accessible sans connexion (consultation anonyme autorisee)
- [ ] La liste est paginee (20 événements par page) avec chargement infini sur mobile
- [ ] Les événements complets (plus de billets disponibles) sont clairement marques

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** Aucune

---

### US-002 : Rechercher un événement par mot-cle

**En tant qu’** acheteur de billets
**Je veux** rechercher un événement par son nom, son artiste ou son lieu
**Afin de** trouver rapidement un événement precis dont j’ai entendu parler

**Critères d’acceptation :**
- [ ] Un champ de recherche est disponible en haut de la liste
- [ ] La recherche s’effectue sur le nom de l’événement, le nom de l’artiste et le nom du lieu
- [ ] Les resultats s’affichent en moins de 2 secondes
- [ ] Un message explicite s’affiche si aucun resultat ne correspond
- [ ] La recherche tolere les fautes de frappe mineures (recherche insensible a la casse)

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-001

---

### US-003 : Filtrer les événements par categorie

**En tant qu’** acheteur de billets
**Je veux** filtrer les événements par type (Concert, Festival, Theatre, Exposition, Sport, Autre)
**Afin de** voir uniquement les événements qui correspondent a mes gouts

**Critères d’acceptation :**
- [ ] Les categories sont accessibles en un clic depuis la liste principale
- [ ] Plusieurs categories peuvent etre selectionnees simultanement
- [ ] Le nombre d’événements disponibles par categorie est affiche
- [ ] Le filtre actif est visuellement distinct
- [ ] La reinitialisation des filtres est possible en un clic

**Priorite :** Must Have
**Complexite :** 2 points
**Dependances :** US-001

---

### US-004 : Filtrer les événements par ville

**En tant qu’** acheteur de billets
**Je veux** filtrer les événements par ville (Cotonou, Porto-Novo, Parakou, etc.)
**Afin de** ne voir que les événements accessibles depuis mon lieu de residence

**Critères d’acceptation :**
- [ ] Un filtre par ville est disponible sur la liste
- [ ] La liste des villes disponibles correspond aux villes ayant des événements actifs
- [ ] Le filtre par ville se combine avec les autres filtres
- [ ] La ville selectionnee est sauvegardee entre les sessions (si connecte)

**Priorite :** Must Have
**Complexite :** 2 points
**Dependances :** US-001

---

### US-005 : Filtrer les événements par date

**En tant qu’** acheteur de billets
**Je veux** filtrer les événements par periode (ce week-end, ce mois, choisir des dates)
**Afin de** voir uniquement les événements qui correspondent a mes disponibilites

**Critères d’acceptation :**
- [ ] Des raccourcis sont disponibles : “Ce week-end”, “Cette semaine”, “Ce mois”
- [ ] Un selecteur de dates personnalise est disponible (date de debut et fin)
- [ ] Le filtre par date se combine avec les autres filtres
- [ ] Les événements deja passes ne sont pas affiches par defaut

**Priorite :** Should Have
**Complexite :** 3 points
**Dependances :** US-001

---

### US-006 : Consulter le detail d’un événement

**En tant qu’** acheteur de billets
**Je veux** voir la page complete d’un événement
**Afin de** obtenir toutes les informations necessaires avant d’acheter mon billet

**Critères d’acceptation :**
- [ ] La page affiche : nom, description complete, date et heure, lieu avec adresse, plan d’acces
- [ ] La page affiche les categories de billets disponibles avec leurs prix
- [ ] La page affiche le nombre de billets restants par categorie (si inferieur a 20)
- [ ] La page affiche les photos ou visuels de l’événement (min. 1 image)
- [ ] La page affiche les informations de l’organisateur
- [ ] Un bouton “Acheter” est accessible en permanence (sticky en bas de page sur mobile)
- [ ] La page est accessible sans connexion

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-001

---

### US-007 : Partager un événement

**En tant qu’** acheteur de billets
**Je veux** partager un événement avec mes contacts via WhatsApp ou en copiant le lien
**Afin de** inviter mes amis a m’accompagner

**Critères d’acceptation :**
- [ ] Un bouton de partage est disponible sur la page detail de l’événement
- [ ] Le partage via WhatsApp est propose en premier (contexte Benin)
- [ ] La copie du lien direct est disponible
- [ ] Le lien partage pointe directement vers la page de l’événement

**Priorite :** Should Have
**Complexite :** 1 point
**Dependances :** US-006

---

# Epic 2 : Compte et profil acheteur

> Surface : A — Application acheteur
> 

### US-008 : Creer un compte acheteur

**En tant que** visiteur
**Je veux** creer un compte acheteur avec mon numero de telephone ou mon email
**Afin de** pouvoir acheter des billets et retrouver mes achats

**Critères d’acceptation :**
- [ ] L’inscription est possible via email + mot de passe ou numero de telephone + mot de passe
- [ ] L’email est obligatoire dans les deux cas
- [ ] Le numero de telephone doit etre un numero beninois valide (+229) s’il est renseigne
- [ ] Un lien de confirmation est envoye par email apres inscription
- [ ] Le compte est actif uniquement apres confirmation via le lien email
- [ ] Le lien de confirmation expire apres 24 heures
- [ ] Les informations minimales requises sont : prenom, nom, email, mot de passe
- [ ] Un message d’erreur clair s’affiche si l’email est deja utilise

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** Aucune

---

### US-009 : Se connecter a son compte

**En tant qu’** acheteur inscrit
**Je veux** me connecter avec mon telephone (OTP) ou mon email et mot de passe
**Afin de** acceder a mes billets et a mon historique d’achats

**Critères d’acceptation :**
- [ ] La connexion par email + mot de passe est disponible
- [ ] La connexion par numero de telephone + mot de passe est disponible (si telephone renseigne au profil)
- [ ] La session est maintenue 30 jours sur l’appareil (token persistant)
- [ ] Un message d’erreur generique s’affiche en cas de credentials incorrects (securite)
- [ ] La redirection s’effectue vers la page precedant la connexion (si achat en cours)

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-008

---

### US-010 : Reinitialiser son mot de passe

**En tant qu’** acheteur inscrit
**Je veux** reinitialiser mon mot de passe via mon email ou mon telephone
**Afin de** retrouver l’acces a mon compte si j’ai oublie mon mot de passe

**Critères d’acceptation :**
- [ ] Une option “Mot de passe oublie” est visible sur l’ecran de connexion
- [ ] La reinitialisation s’effectue par email (lien de reinitialisation)
- [ ] Le lien de reinitialisation expire apres 30 minutes
- [ ] Le nouveau mot de passe doit comporter au moins 8 caracteres

**Priorite :** Must Have
**Complexite :** 2 points
**Dependances :** US-009

---

### US-011 : Consulter et modifier son profil

**En tant qu’** acheteur inscrit
**Je veux** consulter et modifier mes informations personnelles
**Afin de** maintenir mes donnees a jour (nom, email, telephone)

**Critères d’acceptation :**
- [ ] La page profil affiche : prenom, nom, email, telephone
- [ ] Chaque champ est modifiable individuellement
- [ ] La modification de l’email ou du telephone necessite une verification OTP
- [ ] Une confirmation de sauvegarde s’affiche apres modification

**Priorite :** Should Have
**Complexite :** 2 points
**Dependances :** US-009

---

### US-012 : Supprimer son compte

**En tant qu’** acheteur inscrit
**Je veux** supprimer mon compte et mes donnees personnelles
**Afin de** exercer mon droit a l’effacement

**Critères d’acceptation :**
- [ ] L’option de suppression est accessible depuis les parametres du profil
- [ ] Une confirmation est demandee avant suppression (saisie du mot de passe)
- [ ] Les billets deja achetes restent valides meme apres suppression du compte
- [ ] Les donnees personnelles sont anonymisees dans les 30 jours

**Priorite :** Could Have
**Complexite :** 3 points
**Dependances :** US-009

---

# Epic 3 : Achat de billets

> Surface : A — Application acheteur
> 

### US-013 : Selectionner des billets et les quantites

**En tant qu’** acheteur connecte
**Je veux** selectionner le type et la quantite de billets que je veux acheter
**Afin de** preparer ma commande avant de payer

**Critères d’acceptation :**
- [ ] Toutes les categories de billets disponibles sont affichees avec leur prix
- [ ] L’acheteur peut choisir la quantite de billets par categorie (min. 1, max. 10 par commande)
- [ ] Les billets indisponibles (sold out) sont affiches mais non selectionnables
- [ ] Le recapitulatif (sous-total et total) se met a jour dynamiquement
- [ ] L’utilisateur non connecte est redirige vers la connexion avant de continuer

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-006, US-009

---

### US-014 : Payer par Mobile Money (MTN MoMo)

**En tant qu’** acheteur
**Je veux** payer mes billets via MTN Mobile Money
**Afin de** regler ma commande avec le moyen de paiement que j’utilise au quotidien

**Critères d’acceptation :**
- [ ] Le paiement MTN MoMo est propose sur l’ecran de paiement
- [ ] L’acheteur saisit son numero MTN MoMo (pre-rempli avec son numero de compte si disponible)
- [ ] Une notification push est envoyee sur le telephone de l’acheteur pour validation
- [ ] La commande est confirmee uniquement apres validation effective du paiement
- [ ] Un message d’echec clair s’affiche si le solde est insuffisant ou si le delai expire
- [ ] Le paiement expire apres 5 minutes d’inactivite

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-013

---

### US-015 : Payer par Mobile Money (Moov Money)

**En tant qu’** acheteur
**Je veux** payer mes billets via Moov Money
**Afin de** regler ma commande si j’utilise Moov comme operateur

**Critères d’acceptation :**
- [ ] Le paiement Moov Money est propose sur l’ecran de paiement
- [ ] Le flux de paiement suit la meme logique que MTN MoMo (US-014)
- [ ] L’ecran indique clairement quel operateur est selectionne

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-014

---

### US-016 : Payer par carte bancaire

**En tant qu’** acheteur
**Je veux** payer mes billets par carte bancaire (Visa, Mastercard)
**Afin de** regler ma commande si je n’utilise pas le Mobile Money

**Critères d’acceptation :**
- [ ] Le paiement carte est propose sur l’ecran de paiement
- [ ] La saisie des donnees carte se fait via un formulaire securise (PCI DSS)
- [ ] La confirmation de paiement s’affiche en moins de 10 secondes
- [ ] Les donnees de carte ne sont jamais stockees par Wamiton

**Priorite :** Should Have
**Complexite :** 5 points
**Dependances :** US-013

---

US-014 à US-016 peuvent être gérés par un même agrégateur de paiements 

---

### US-017 : Recevoir une confirmation de commande

**En tant qu’** acheteur ayant paye
**Je veux** recevoir une confirmation de ma commande avec mes billets
**Afin de** avoir la preuve de mon achat immediatement

**Critères d’acceptation :**
- [ ] Un ecran de confirmation s’affiche immediatement apres paiement valide
- [ ] Un email de confirmation est envoye apres paiement valide
- [ ] La confirmation contient : recapitulatif de commande, numero de commande, et acces aux billets
- [ ] Les billets sont accessibles dans l’application moins de 30 secondes apres paiement

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-014, US-015

---

### US-018 : Recevoir un billet avec QR code a l’emission

**En tant qu’** acheteur ayant paye
**Je veux** recevoir mon billet numerique avec un QR code unique au moment de l’emission definie par l’organisateur
**Afin de** presenter ce billet a l’entree de l’événement avec un maximum de securite

**Critères d’acceptation :**
- [ ] Apres achat, l’acheteur recoit une confirmation de commande sans QR code (billet en attente d’emission)
- [ ] Le QR code est genere et envoye par email selon le delai d’emission configure par l’organisateur (par defaut 24h avant l’événement)
- [ ] Chaque QR code est unique, lie a un billet et non reutilisable
- [ ] Le billet emis affiche : nom de l’événement, date, lieu, categorie, nom du detenteur actuel
- [ ] Un QR code ne peut etre scanne qu’une seule fois (premier scan valide, les suivants refuses)
- [ ] Le billet emis est envoye par email et par WhatsApp (si numero renseigne au profil) au moment de l’emission
- [ ] L’acheteur recoit une notification in-app, un email et un message WhatsApp au moment de l’emission
- [ ] Le statut du billet dans l’application passe de “En attente d’emission” a “Emis” au moment de l’emission

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-017, US-040

---

# Epic 4 : Gestion des billets achetes

> Surface : A — Application acheteur
> 

### US-019 : Consulter mes billets

**En tant qu’** acheteur connecte
**Je veux** voir tous mes billets (à venir et passes) avec leur statut d’emission
**Afin de** suivre l’etat de mes achats et acceder a mes QR codes quand ils sont disponibles

**Critères d’acceptation :**
- [ ] Un onglet “Mes billets” est accessible depuis le menu principal
- [ ] Les billets sont classes en deux sections : “A venir” et “Passes”
- [ ] Chaque billet affiche : nom de l’événement, date, lieu, categorie, statut (En attente d’emission / Emis / Cede / Utilise)
- [ ] Le QR code est accessible en un clic uniquement apres emission
- [ ] Un message explicite indique la date d’emission prevue pour les billets en attente
- [ ] Les billets emis sont accessibles sans connexion internet (cache local)

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-018

---

### US-020 : Consulter l’historique de mes commandes

**En tant qu’** acheteur connecte
**Je veux** voir l’historique de toutes mes commandes
**Afin de** suivre mes depenses et retrouver les details d’un achat passe

**Critères d’acceptation :**
- [ ] L’historique est accessible depuis le profil
- [ ] Chaque commande affiche : date d’achat, événement, montant paye, statut, numero de commande
- [ ] Le detail complet d’une commande est accessible au clic
- [ ] Les commandes echouees ou annulees sont incluses avec leur statut

**Priorite :** Must Have
**Complexite :** 2 points
**Dependances :** US-019

---

### US-021 : Afficher mon billet en mode hors connexion

**En tant qu’** acheteur
**Je veux** acceder a mon billet et son QR code sans avoir besoin d’internet
**Afin de** presenter mon billet a l’entree meme si la connexion est mauvaise sur le lieu de l’événement

**Critères d’acceptation :**
- [ ] Les billets emis sont mis en cache local lors du dernier acces connecte
- [ ] Le QR code est affichable offline a partir des donnees en cache
- [ ] Un indicateur visuel signale que le billet est affiche en mode hors connexion
- [ ] Le billet hors connexion reste valide pour le scan du jour de l’événement
- [ ] Le billet imprimable (PDF) genere via US-042 est egalement accepte a l’entree

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-018, US-042

---

### US-038 : Ceder un billet a un autre utilisateur

**En tant qu’** acheteur detenteur d’un billet non encore emis
**Je veux** ceder mon billet a un autre utilisateur Wamiton
**Afin de** lui transferer mon droit d’entree si je ne peux pas assister a l’événement

**Critères d’acceptation :**
- [ ] La cession est possible uniquement si le billet n’est pas encore emis (statut “En attente d’emission”)
- [ ] La cession est possible uniquement avant la deadline de cession definie par l’organisateur
- [ ] L’acheteur recherche le cessionnaire par email ou numero de telephone enregistre sur Wamiton
- [ ] Le cessionnaire doit avoir un compte Wamiton actif pour recevoir le billet
- [ ] Une confirmation est demandee au cedant avant de finaliser la cession
- [ ] Le cedant recoit un email confirmant la cession et perd l’acces au billet immediatement
- [ ] Un billet deja cede ne peut pas etre cede a nouveau par le nouveau detenteur avant emission
- [ ] La cession est gratuite pour cette version (pas de frais supplementaires)

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-018, US-040

---

### US-039 : Recevoir un billet cede

**En tant qu’** utilisateur Wamiton
**Je veux** recevoir un billet qu’un autre utilisateur me cede
**Afin de** pouvoir assister a l’événement a sa place

**Critères d’acceptation :**
- [ ] Le cessionnaire recoit une notification in-app et un email l’informant de la cession
- [ ] Le billet apparait immediatement dans sa liste “Mes billets” avec le statut “En attente d’emission”
- [ ] Le billet affiche le nom du cessionnaire (nouveau detenteur) et non celui du cedant initial
- [ ] A l’emission, le QR code genere contient les informations du cessionnaire
- [ ] Le cessionnaire peut a son tour ceder le billet avant emission si la deadline n’est pas depassee

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-038

---

### US-042 : Telecharger et imprimer son billet en PDF

**En tant qu’** acheteur dont le billet est emis
**Je veux** telecharger mon billet en PDF pour l’imprimer
**Afin de** presenter un billet papier a l’entree si je n’ai pas acces a mon smartphone

**Critères d’acceptation :**
- [ ] Le telechargement PDF est disponible uniquement apres emission du billet
- [ ] Le PDF contient : nom de l’événement, date, lieu, categorie, nom du detenteur, QR code en haute resolution
- [ ] Le PDF est genere en moins de 5 secondes
- [ ] Le QR code sur le PDF est identique a celui de l’application (meme logique de validation unique)
- [ ] Le PDF est envoye par email et par WhatsApp (si numero renseigne au profil) au moment de l’emission

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-018

---

# Epic 5 : Compte et profil organisateur

> Surface : B — Tableau de bord organisateur
> 

### US-022 : Creer un compte organisateur

**En tant que** promoteur ou association culturelle
**Je veux** creer un compte organisateur
**Afin de** publier mes événements et vendre mes billets sur Wamiton

**Critères d’acceptation :**
- [ ] Un formulaire d’inscription organisateur est distinct du formulaire acheteur
- [ ] Les informations requises sont : nom commercial, email, contact principal (telephone + email), ville
- [ ] L’email est obligatoire et doit etre confirme via un lien envoye par email avant soumission
- [ ] Le compte organisateur est soumis a une validation manuelle par Wamiton avant activation
- [ ] L’organisateur recoit un email quand son compte est valide
- [ ] Un compte acheteur existant peut etre converti en compte organisateur

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-008

---

### US-023 : Completer son profil organisateur

**En tant qu’** organisateur inscrit
**Je veux** completer mon profil avec mes informations et mon logo
**Afin de** inspirer confiance aux acheteurs et donner de la visibilite a ma marque

**Critères d’acceptation :**
- [ ] Le profil organisateur contient : nom commercial, description, logo, telephone, email, reseaux sociaux
- [ ] Le logo accepte les formats JPEG et PNG (max 2 MB)
- [ ] Le profil est visible sur les pages d’événements publies par l’organisateur
- [ ] Les modifications sont effectives immediatement

**Priorite :** Should Have
**Complexite :** 2 points
**Dependances :** US-022

---

# Epic 6 : Gestion des événements

> Surface : B — Tableau de bord organisateur
> 

### US-024 : Creer un événement

**En tant qu’** organisateur
**Je veux** publier un événement sur Wamiton
**Afin de** le rendre visible a tous les acheteurs potentiels et vendre des billets en ligne

**Critères d’acceptation :**
- [ ] Le formulaire de creation contient les champs obligatoires : nom, description, categorie, date/heure debut, ville, lieu (nom + adresse)
- [ ] L’organisateur peut ajouter jusqu’a 5 photos (JPEG/PNG, max 5 MB chacune)
- [ ] Une image de couverture principale est obligatoire
- [ ] L’événement est soumis a une validation par l’equipe Wamiton avant publication
- [ ] L’organisateur recoit une notification quand l’événement est valide et publie
- [ ] Un brouillon peut etre sauvegarde et repris plus tard

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-022

---

### US-025 : Configurer les categories de billets

**En tant qu’** organisateur creant un événement
**Je veux** definir les categories de billets (Early Bird, Standard, VIP) avec leur prix et leur quota
**Afin de** proposer plusieurs options d’achat et controler le nombre de places vendues

**Critères d’acceptation :**
- [ ] L’organisateur peut creer jusqu’a 5 categories de billets par événement
- [ ] Chaque categorie requiert : nom, prix (FCFA), nombre de places disponibles
- [ ] Le prix peut etre 0 FCFA (événement gratuit, billet d’entree numerique tout de meme genere)
- [ ] Une date de fin de vente peut etre definie par categorie (ex : Early Bird expire le X)
- [ ] Le total des places de toutes les categories constitue la capacite totale de l’événement

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-024

---

### US-026 : Modifier un événement publie

**En tant qu’** organisateur
**Je veux** modifier les informations d’un événement deja publie
**Afin de** corriger une erreur ou mettre a jour des informations (changement de lieu, d’horaire)

**Critères d’acceptation :**
- [ ] Les champs modifiables apres publication sont : description, photos, horaire, lieu
- [ ] La modification du nom de l’événement necessite une revalidation par Wamiton
- [ ] Les acheteurs ayant deja achete un billet sont notifies par SMS si la date ou le lieu change
- [ ] Le prix et les categories de billets ne peuvent pas etre modifies si des billets sont deja vendus

**Priorite :** Should Have
**Complexite :** 5 points
**Dependances :** US-024

---

### US-027 : Annuler un événement

**En tant qu’** organisateur
**Je veux** annuler un événement publie
**Afin de** informer les acheteurs et declencher les remboursements en cas d’empechement

**Critères d’acceptation :**
- [ ] L’annulation est possible depuis le tableau de bord de l’événement
- [ ] Une confirmation est demandee avec saisie d’un motif d’annulation
- [ ] Tous les acheteurs sont notifies par email dans les 30 minutes suivant l’annulation
- [ ] Le remboursement des billets est declenche automatiquement sur le moyen de paiement initial
- [ ] L’événement annule reste visible sur la plateforme avec le statut “Annule”

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-024

---

### US-028 : Consulter la liste de ses événements

**En tant qu’** organisateur
**Je veux** voir tous mes événements (en attente, publies, passes, annules)
**Afin de** gerer mon catalogue d’événements facilement

**Critères d’acceptation :**
- [ ] La liste des événements est accessible depuis le tableau de bord organisateur
- [ ] Les événements sont filtres par statut : Brouillon, En attente, Publie, Passe, Annule
- [ ] Les indicateurs cles sont visibles sur chaque événement : billets vendus / disponibles, recette totale
- [ ] Un raccourci vers la gestion du check-in est disponible pour les événements du jour

**Priorite :** Must Have
**Complexite :** 3 points
**Dependances :** US-024

---

### US-040 : Configurer les parametres de billetterie d’un événement

**En tant qu’** organisateur
**Je veux** definir les parametres temporels de la billetterie de mon événement
**Afin de** controler la fermeture des ventes, l’emission des billets et la deadline de cession

**Critères d’acceptation :**
- [ ] L’organisateur peut definir la fermeture de la billetterie en ligne (X heures avant le debut, defaut : 2h)
- [ ] L’organisateur peut definir le delai d’emission des billets (X heures avant le debut, defaut : 24h)
- [ ] L’organisateur peut definir la deadline de cession (automatiquement calquee sur l’emission par defaut)
- [ ] Les trois parametres sont configures lors de la creation de l’événement et modifiables jusqu’a la fermeture de la billetterie
- [ ] Un recapitulatif des dates cles est affiche : “Ventes ferment le [date], Billets emis le [date], Cession possible jusqu’au [date]”
- [ ] Ces dates cles sont affichees aux acheteurs sur la page de l’événement et dans leurs billets
- [ ] Une alerte s’affiche si le delai d’emission est superieur au delai de fermeture de billetterie (incoherence)

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-024

---

# Epic 7 : Vente et billetterie en ligne

> Surface : B — Tableau de bord organisateur
> 

### US-029 : Suivre les ventes en temps reel

**En tant qu’** organisateur
**Je veux** suivre les ventes de billets en temps reel pour mon événement
**Afin de** adapter ma communication et anticiper le remplissage de la salle

**Critères d’acceptation :**
- [ ] Le tableau de bord affiche : nombre de billets vendus par categorie, recette totale, taux de remplissage
- [ ] Les donnees sont actualisees au minimum toutes les 5 minutes
- [ ] Un graphique des ventes dans le temps est disponible (ventes par jour)
- [ ] Une alerte est envoyee a l’organisateur quand un seuil est atteint (ex : 80% de la capacite)

**Priorite :** Should Have
**Complexite :** 5 points
**Dependances :** US-028

---

### US-030 : Exporter la liste des acheteurs

**En tant qu’** organisateur
**Je veux** exporter la liste des acheteurs de billets de mon événement
**Afin de** avoir une liste d’entree de secours et archiver les donnees de l’événement

**Priorite :** Should Have
**Complexite :** 2 points
**Dependances :** US-029

**Critères d’acceptation :**
- [ ] L’export est disponible au format CSV
- [ ] L’export peut contenir : prenom, nom, telephone, categorie de billet, nombre de billets, statut du billet
- [ ] L’export peut etre filtre par categorie de billet
- [ ] L’export est disponible a tout moment (avant, pendant et apres l’événement)

La sensibilité des informations reste à définir

---

### US-031 : Configurer les frais de service

**En tant qu’** organisateur
**Je veux** savoir exactement combien je vais recevoir par billet vendu
**Afin de** definir mes prix en tenant compte des commissions de la plateforme

**Critères d’acceptation :**
- [ ] Les frais de service Wamiton sont clairement affiches lors de la creation des categories de billets
- [ ] Le simulateur de recette affiche : prix billet / frais Wamiton / montant reverse
- [ ] Les frais sont deduites automatiquement lors du reversement des fonds

**Priorite :** Must Have
**Complexite :** 2 points
**Dependances :** US-025

---

### US-032 : Recevoir le reversement des recettes

**En tant qu’** organisateur
**Je veux** recevoir le reversement des recettes de vente de billets
**Afin de** percevoir les fonds generes par mon événement

**Critères d’acceptation :**
- [ ] Le reversement est effectue sur le compte Mobile Money de l’organisateur (MTN MoMo ou Moov Money)
- [ ] Le reversement s’effectue apres la tenue de l’événement (delai de 48h apres la date de l’événement)
- [ ] L’organisateur recoit une notification de virement avec le montant detaille
- [ ] Un historique des reversements est disponible dans le tableau de bord

**Priorite :** Must Have
**Complexite :** 5 points
**Dependances :** US-031

---

# Epic 8 : Controle des entrees (Check-in)

> Surface : C — Application check-in
> 

### US-033 : Scanner un billet QR code a l’entree

**En tant qu’** agent de controle d’un organisateur
**Je veux** scanner le QR code d’un billet depuis mon smartphone
**Afin de** valider l’entree d’un participant rapidement et eviter les fraudes

**Critères d’acceptation :**
- [ ] Le scan est disponible depuis l’application check-in (Surface C)
- [ ] Le scan fonctionne avec la camera du smartphone sans materiel supplementaire
- [ ] La validation s’affiche en moins de 2 secondes apres scan (connexion normale)
- [ ] Un affichage vert confirme un billet valide (avec nom du participant et categorie)
- [ ] Un affichage rouge bloque un billet deja utilise ou invalide
- [ ] Le scan fonctionne en mode hors connexion en s’appuyant sur la liste synchronisee (US-041)

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-018, US-022, US-041

---

### US-041 : Synchroniser la liste des billets pour le scan hors connexion

**En tant qu’** agent de controle ou organisateur
**Je veux** telecharger la liste complete des billets valides de mon événement avant d’arriver sur site
**Afin de** pouvoir scanner les billets sans connexion internet le jour de l’événement

**Critères d’acceptation :**
- [ ] La synchronisation est disponible des la fermeture de la billetterie en ligne (US-040)
- [ ] La liste synchronisee contient tous les QR codes valides et leur statut au moment du telechargement
- [ ] La synchronisation est declenchee manuellement par l’agent depuis l’application check-in
- [ ] La taille et la date de la derniere synchronisation sont affichees clairement
- [ ] En mode hors connexion, chaque scan met a jour le statut localement (utilise / non utilise)
- [ ] A la reconnexion, les scans effectues hors connexion sont synchronises avec le serveur
- [ ] Un avertissement s’affiche si la liste n’a pas ete synchronisee depuis plus de 2 heures

**Priorite :** Must Have
**Complexite :** 8 points
**Dependances :** US-033, US-040

---

### US-034 : Gerer plusieurs agents de controle

**En tant qu’** organisateur
**Je veux** permettre a plusieurs membres de mon equipe de scanner les billets simultanement
**Afin de** fluidifier les entrees en ouvrant plusieurs files d’acces

**Critères d’acceptation :**
- [ ] L’organisateur peut creer des comptes agents temporaires (acces uniquement au scan de son événement)
- [ ] Plusieurs agents peuvent scanner en meme temps sans conflits (pas de double validation du meme billet)
- [ ] L’acces agent expire automatiquement apres la date de l’événement
- [ ] L’organisateur voit en temps reel le nombre de personnes entrees

**Priorite :** Should Have
**Complexite :** 5 points
**Dependances :** US-033

---

### US-035 : Consulter les statistiques d’entrees en temps reel

**En tant qu’** organisateur
**Je veux** voir le nombre de participants entres en temps reel pendant mon événement
**Afin de** adapter la logistique (securite, bar, scene) en fonction de la progression

**Critères d’acceptation :**
- [ ] Le tableau de bord affiche : entrees totales, entrees par categorie de billet, pourcentage de remplissage
- [ ] Les donnees sont actualisees toutes les 30 secondes
- [ ] Un graphique d’affluence dans le temps est disponible
- [ ] L’historique des scans est accessible (heure de chaque entree)

**Priorite :** Could Have
**Complexite :** 3 points
**Dependances :** US-033

---

# Epic 9 : Tableau de bord organisateur

> Surface : B — Tableau de bord organisateur
> 

### US-036 : Acceder a un tableau de bord synthetique

**En tant qu’** organisateur
**Je veux** voir un tableau de bord avec les indicateurs cles de mes événements
**Afin de** piloter mon activite en un coup d’oeil

**Critères d’acceptation :**
- [ ] Le tableau de bord affiche : événements actifs, total de billets vendus ce mois, recettes en cours
- [ ] Les événements à venir sont listes avec leur taux de remplissage
- [ ] Les événements qui approchent de leur date (7 jours) sont mis en evidence
- [ ] Un acces rapide aux fonctions les plus utilisees est disponible (creer événement, scanner)

**Priorite :** Should Have
**Complexite :** 3 points
**Dependances :** US-028, US-029

---

### US-037 : Recevoir des notifications sur les ventes et l’activite

**En tant qu’** organisateur
**Je veux** recevoir des notifications push et SMS sur les actions importantes
**Afin de** etre tenu informe sans avoir a consulter l’application en permanence

**Critères d’acceptation :**
- [ ] Une notification est envoyee a chaque vente de billet (configurable : immediat ou recap quotidien)
- [ ] Une alerte est envoyee quand un palier de vente est atteint (50%, 80%, sold out)
- [ ] Une notification rappelle l’événement 24h avant la date
- [ ] Les notifications peuvent etre desactivees par type depuis les parametres

**Priorite :** Could Have
**Complexite :** 3 points
**Dependances :** US-028

---

---

# Recapitulatif par priorite

> Surface A : Application acheteur (wamiton.bj)
Surface B : Tableau de bord organisateur (dashboard.wamiton.bj)
Surface C : Application check-in (checkin.wamiton.bj)
> 

### Must Have (MVP) - 29 stories

| US | Titre | Surface |
| --- | --- | --- |
| US-001 | Consulter la liste des événements à venir | A |
| US-002 | Rechercher un événement par mot-cle | A |
| US-003 | Filtrer les événements par categorie | A |
| US-004 | Filtrer les événements par ville | A |
| US-006 | Consulter le detail d’un événement | A |
| US-008 | Creer un compte acheteur | A |
| US-009 | Se connecter a son compte | A |
| US-010 | Reinitialiser son mot de passe | A |
| US-013 | Selectionner des billets et les quantites | A |
| US-014 | Payer par Mobile Money (MTN MoMo) | A |
| US-015 | Payer par Mobile Money (Moov Money) | A |
| US-017 | Recevoir une confirmation de commande | A |
| US-018 | Recevoir un billet avec QR code a l’emission | A |
| US-019 | Consulter mes billets | A |
| US-020 | Consulter l’historique de mes commandes | A |
| US-021 | Afficher mon billet en mode hors connexion | A |
| US-038 | Ceder un billet a un autre utilisateur | A |
| US-039 | Recevoir un billet cede | A |
| US-042 | Telecharger et imprimer son billet en PDF | A |
| US-022 | Creer un compte organisateur | B |
| US-024 | Creer un événement | B |
| US-025 | Configurer les categories de billets | B |
| US-027 | Annuler un événement | B |
| US-028 | Consulter la liste de ses événements | B |
| US-031 | Configurer les frais de service | B |
| US-032 | Recevoir le reversement des recettes | B |
| US-040 | Configurer les parametres de billetterie | B |
| US-033 | Scanner un billet QR code a l’entree | C |
| US-041 | Synchroniser la liste des billets pour le scan offline | C |

### Should Have (Phase 2) - 10 stories

| US | Titre | Surface |
| --- | --- | --- |
| US-005 | Filtrer les événements par date | A |
| US-007 | Partager un événement | A |
| US-011 | Consulter et modifier son profil | A |
| US-016 | Payer par carte bancaire | A |
| US-023 | Completer son profil organisateur | B |
| US-026 | Modifier un événement publie | B |
| US-029 | Suivre les ventes en temps reel | B |
| US-030 | Exporter la liste des acheteurs | B |
| US-034 | Gerer plusieurs agents de controle | C |
| US-036 | Acceder a un tableau de bord synthetique | B |

### Could Have (Phase 3) - 3 stories

| US | Titre | Surface |
| --- | --- | --- |
| US-012 | Supprimer son compte | A |
| US-035 | Consulter les statistiques d’entrees en temps reel | C |
| US-037 | Recevoir des notifications sur les ventes et l’activite | B |

### Won’t Have (Hors scope V1)

- Revente de billets entre particuliers (la cession gratuite couvre ce besoin en V1)
- Abonnements et formules multi-événements
- Extension multi-pays (UEMOA) — Phase 3+
- Programme de fidelite acheteur
- Application mobile native (iOS / Android) — PWA en V1