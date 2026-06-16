# mon projet - Cameroun" 
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

# 10. Stack technique
##a

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
