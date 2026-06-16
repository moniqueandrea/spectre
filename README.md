# CAHIER DES CHARGES FONCTIONNEL & TECHNIQUE
## Projet : SPECTRE
Sous-titre : Système de Pilotage Évolué du Cycle de vie et du Traitement des Réparations d’Équipements
Thématique Cohorte : Cameroun 2030 — Le numérique au service du quotidien
Livrable : Fiche d'Atelier (Section 4) — Projet Individuel
Statut : Spécifications Validées (Zéro Faille Logique)
### 1. Titre du projet
SPECTRE — L'application web progressive et légère dédiée à la traçabilité des actifs informatiques et à l’automatisation de la maintenance interne pour les PME.
### 2. Le problème concret
Dans l'écosystème des PME et agences de services à Douala, la gestion des immobilisations technologiques (ordinateurs portables, écrans, smartphones de fonction) repose presque exclusivement sur l'informel : registres physiques ou tableurs Excel locaux corrompus.
Cette absence de rigueur engendre trois problèmes critiques au quotidien :
 * Perte de traçabilité matérielle : Difficulté à savoir avec certitude quel employé détient quel équipement à un instant T, générant des litiges internes ou des pertes d'actifs non documentées lors des départs de personnel.
 * Asymétrie d'information sur les pannes : Les anomalies techniques sont remontées de manière verbale ou par message WhatsApp. Le technicien, souvent surchargé ou en déplacement, oublie l'incident, n'a aucun historique écrit de la panne, ce qui retarde l'intervention et paralyse la productivité des employés.
 * Paralysie par coupure réseau : Les solutions de gestion classiques du marché sont des SaaS gourmands en bande passante. Dès que la connexion internet faiblit ou coupe (comme c'est fréquemment le cas), les équipes ne peuvent plus ni déclarer d'incidents ni mettre à jour l'inventaire.
### 3. Cible (Utilisateurs spécifiques)
 * Les Collaborateurs / Employés de l'entreprise : Personnels de tous les départements (Finance, RH, Opérations) qui utilisent un équipement de fonction et doivent pouvoir signaler instantanément une anomalie technique.
 * Le Responsable Logistique / Technicien IT interne : L'administrateur de l'application qui pilote le statut du parc, affecte le matériel disponible aux nouvelles recrues, et gère la file d'attente des réparations.
### 4. Proposition de valeur
SPECTRE apporte une visibilité totale (comme un "spectre" qui voit tout) sur le cycle de vie du matériel d'une entreprise. Grâce à son architecture *Offline-First*, elle garantit aux structures de Douala un outil de suivi immuable, fluide et parfaitement opérationnel, même au cœur des zones à connectivité instable (ex: micro-coupures de réseau à Yassa), augmentant la durée de vie des machines et éliminant les temps morts administratifs.
### 5. Analyse des Risques et Parades Étanches (Zéro Faille)
Pour garantir une application industrielle exempte de failles, SPECTRE intègre des contre-mesures architecturales dès sa conception :
 * Risque de Conflit de Synchronisation (Deux managers modifient un statut hors-ligne simultanément) :
   * *Impact :* Écrasement mutuel des données lors du retour au réseau.
   * *Parade SPECTRE :* Chaque entité possède un horodatage strict (lastUpdatedAt) et un état de synchronisation (syncStatus). En cas de retour en ligne, le service Angular compare les timestamps et applique une stratégie de résolution intelligente (alerte contextuelle ou priorité aux données du serveur).
 * Risque de Saturation du Stockage Local (Capacité du localStorage limitée à 5 Mo) :
   * *Impact :* Crash de l'application ou refus d'enregistrement des saisies utilisateurs.
   * *Parade SPECTRE :* Pratique stricte de la minimisation des données (*Data Minimization*). Seuls les index actifs, le personnel et les tickets non résolus sont persistés en local. Une routine de purge automatique supprime l'historique local des tickets clos de plus de 45 jours dès que la synchronisation serveur
   ** *Parade SPECTRE :* Implémentation d'une machine d'état immuable au niveau des services Angular. Un matériel au statut UNDER_REPAIR ou DECOMMISSIONED rejette algorithmiquement toute tentative d'affectation ou de transfert de responsabilité.
 * Risque de Vol de Session / Curiosité (Navigateur laissé ouvert sur un poste partagé) :
   * *Impact :* Modifications frauduleuses ou malveillantes des attributions d'actifs.
   * *Parade SPECTRE :* Mise en place d'un service d'écoute global (rxjs: fromEvent) surveillant l'activité utilisateur (mouvements de souris, pressions de touches). L'interface utilisateur se verrouille automatiquement (Auto-Lock Guard) après 5 minutes d'inactivité totale.
### 6. Fonctionnalités MVP (Le cœur réactif de l'application)
 * Fonctionnalité 1 : Le Dashboard Analytique
   * Agrégation des données du parc sous forme de KPIs dynamiques mis à jour en temps réel via RxJS : *Total des actifs*, *Taux de disponibilité globale (équipements opérationnels)*, et *Compteur critique des pannes urgentes en attente*.
 * Fonctionnalité 2 : Le Registre d'Inventaire Évolué (CRUD)
   * Liste exhaustive du matériel avec un moteur de recherche instantané basé sur l'identifiant unique constructeur ou le code d’inventaire interne gravé pour interdire toute confusion d'appareil. Formulaire de création sécurisé via les ReactiveForms d'Angular (champs obligatoires, formats de tags imposés).
 * Fonctionnalité 3 : Le Moteur d'Attribution Sécurisé
   * Interface de transfert de responsabilité associant un matériel libre à un employé. Le système intègre un verrouillage au niveau des composants : le bouton "Assigner" est mathématiquement désactivé si le statut du matériel n'est pas strictement égal à AVAILABLE.
 * Fonctionnalité 4 : Le Gestionnaire de Tickets de Maintenance
   * Formulaire de signalement de dysfonctionnement par l'employé (minimum 10 caractères requis). La validation bascule automatiquement l'appareil au statut UNDER_REPAIR. Le ticket suit un cycle de vie strict contrôlé par le technicien (*OPEN -> IN_PROGRESS -> RESOLVED*), exigeant une note de clôture textuelle obligatoire pour libérer et rendre à nouveau disponible le matériel.
### 7. Fonctionnalités bonus (Évolutions optionnelles)
 * Génération de fiche de décharge PDF : Création automatique d’un reçu numérique d'attribution attestant que l'employé a pris possession de l'équipement, prêt pour impression ou signature.
 * Statistiques de robustesse : Graphique analytique mettant en évidence les marques ou modèles de machines qui enregistrent le plus de pannes au sein de l'entreprise pour guider judicieusement les futurs budgets d'achat.
### 8. Écrans / Pages à concevoir
 1. Page de Connexion (Login) : Interface d'authentification propre permettant de simuler la connexion selon le rôle (Admin/Technicien ou Employé standard) via une gestion de session éphémère sécurisée (sessionStorage).
 2. Vue Tableau de Bord : Écran d'accueil présentant les fiches statistiques visuelles et les alertes de maintenance prioritaires.
 3. Vue Inventaire & Gestion : Tableau central de suivi du matériel intégrant les fenêtres modales (pop-ups) pour l'ajout de nouvelles machines et les formulaires d'affectation aux collaborateurs.
 4. Vue Centre de Maintenance : Espace dédié à la liste des incidents, triés par niveau d'urgence, avec des boutons d'action pour faire évoluer l'état des réparations.
### 9. Données manipulées (Structures strictes TypeScript)
Toutes les interfaces de données sont typées de bout en bout dans le dossier src/app/core/models/ pour interdire l'usage du type générique any (exigence stricte du Lab) :
```typescript
export type CategoryMat = 'LAPTOP' | 'SMARTPHONE' | 'MONITOR' | 'NETWORKING' | 'ACCESSORY';
export type StatutMat = 'AVAILABLE' | 'ASSIGNED' | 'UNDER_REPAIR' | 'DECOMMISSIONED';

export interface Material {
  id: string;                    // Identifiant unique universel (UUID v4) pour éviter les conflits d'ID
  serialNumber: string;          // Numéro de série unique constructeur (ex: Tag Dell)
internalInventoryTag: string;  // Tag d'inventaire propre à l'entreprise (ex: SPT-2026-045)
  brand: string;
  model: string;
  category: CategoryMat;
  status: StatutMat;
  assignedToEmployeeId: string | null; // ID de l'employé ou null si l'appareil est en stock
  addedAt: string;               // ISO Date String
  lastUpdatedAt: string;         // ISO Date String pour la résolution des conflits
}

export interface MaintenanceTicket {
  id: string;                    // UUID v4
  materialId: string;            // Clé étrangère pointant vers Material.id
  reportedByEmployeeId: string;  // Clé étrangère pointant vers Employee.id
  description: string;           // Minimum 10 caractères
  priority: 'LOW' | 'MEDIUM' | 'HIGH' | 'CRITICAL';
  status: 'OPEN' | 'IN_PROGRESS' | 'RESOLVED' | 'CANCELLED';
  createdAt: string;             // ISO Date String
  resolvedAt: string | null;     // ISO Date String ou null si actif
  technicianNotes: string | null; // Obligatoire pour passer au statut RESOLVED
  syncStatus: 'SYNCED' | 'PENDING_CREATE' | 'PENDING_UPDATE'; // Flag essentiel pour le mode déconnecté
}

export interface Employee {
  id: string;
  name: string;
  firstName: string;
  department: 'IT' | 'HR' | 'FINANCE' | 'MARKETING' | 'OPERATIONS';
  email: string;
  role: 'ADMIN' | 'TECHNICIAN' | 'USER';
}
``` 

### 10. Stack technique

 * Framework Frontend : Angular 19+ (Architecture moderne basée sur les composants *Standalone*, garantissant la légèreté et la rapidité de chargement des fichiers).
 * Gestion d'état & Asynchronisme : RxJS via l'usage exclusif de BehaviorSubject au sein des services Angular pour maintenir un état applicatif réactif, fluide et immuable.
 * Design & UI : Tailwind CSS pour concevoir une interface utilitaire, épurée, minimaliste au niveau du poids des fichiers et entièrement adaptative (*Mobile-First*).
 * Persistance : Simulation de base de données à travers un fichier de données fictives (Mock JSON) combinée à une sauvegarde automatique dans le cache local du navigateur.
   
### 11. Contrainte connexion (Stratégie Offline-First)
Pour transformer la contrainte de connectivité locale en un atout majeur (le "bon réflexe" demandé par l'atelier) :
 * Persistance applicative : Dès que l'application détecte une perte de signal (via l'écouteur d'état navigator.onLine), toutes les actions d'écriture (création de ticket, changement de statut) dévient leur flux vers le localStorage du navigateur. Les entités reçoivent immédiatement le flag PENDING_CREATE ou PENDING_UPDATE.
 * Indicateur d'état UI résilient : Un composant d'alerte global (NetworkStatusComponent) s'affiche au sommet de l'application. Si le réseau coupe, il passe dynamiquement d'un état "En ligne / Synchronisé" (Vert) à un état "Mode Local Sécurisé — En attente de réseau (X actions en attente)" (Orange) pour rassurer et guider l'utilisateur.
 * Synchronisation automatique en tâche de fond : Dès la reconnative du signal internet, un service applicatif Angular intercepte l'événement online, dépile les actions en attente stockées localement, résout les conflits éventuels par comparaison des dates de modification (lastUpdatedAt), pousse les données vers le store global et repasse le bandeau au vert, de manière totalement transparente pour l'utilisateur.
## 📄 Critères d'Acceptation Final
 1. Le drapeau "strict": true est activé dans le fichier tsconfig.json. Aucun type any n'est toléré.
 2. En simulant une coupure réseau totale dans l'inspecteur du navigateur, l'utilisateur peut créer un ticket et naviguer sans aucun crash de l'interface.
 3. Les boutons d'action se désactivent d'eux-mêmes si la logique métier n'est pas respectée (ex: impossible d'assigner un PC si son statut est UNDER_REPAIR).
