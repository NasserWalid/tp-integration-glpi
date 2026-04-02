# Analyse de la Base de Données GLPI

## Q1. 10 tables principales de GLPI
1. `glpi_tickets` : Stocke toutes les informations des tickets d'assistance.
2. `glpi_computers` : Inventaire des ordinateurs enregistrés.
3. `glpi_users` : Liste des utilisateurs de la plateforme.
4. `glpi_entities` : Gestion multi-entités.
5. `glpi_itilcategories` : Arborescence des catégories de tickets.
6. `glpi_locations` : Liste des lieux physiques.
7. `glpi_profiles` : Gestion des rôles et habilitations.
8. `glpi_groups` : Groupes d'utilisateurs.
9. `glpi_slms` : Accords de niveau de service (SLA).
10. `glpi_logs` : Historique des actions.

## Q2. Nombre de tickets par statut
```sql
SELECT 
  CASE status
    WHEN 1 THEN 'Nouveau'
    WHEN 2 THEN 'En cours (attribué)'
    WHEN 3 THEN 'En cours (planifié)'
    WHEN 4 THEN 'En attente'
    WHEN 5 THEN 'Résolu'
    WHEN 6 THEN 'Clos'
  END AS statut,
  COUNT(*) AS nombre
FROM glpi_tickets
GROUP BY status
ORDER BY status;
```

## Q3. Nombre de tickets créés par mois sur les 12 derniers mois
*(Requête adaptée pour MySQL/MariaDB)*
```sql
SELECT 
    DATE_FORMAT(date_creation, '%Y-%m') AS mois,
    COUNT(*) AS tickets_crees
FROM glpi_tickets
WHERE date_creation >= DATE_SUB(NOW(), INTERVAL 12 MONTH)
GROUP BY mois
ORDER BY mois;
```

## Q4. Équipements associés à un utilisateur donné
```sql
SELECT 
    c.name AS nom_ordinateur, 
    c.serial AS numero_serie, 
    u.name AS identifiant_utilisateur
FROM glpi_computers c
JOIN glpi_users u ON c.users_id = u.id
WHERE u.name = 'glpi';
```

## Q5. Suivi des SLA
Les tables liées sont `glpi_slms` et `glpi_slalevels`.
**Relation :** Dans la table `glpi_tickets`, les champs `slas_id_ttr` et `slas_id_tto` pointent vers les ID des SLAs pour calculer les échéances.
