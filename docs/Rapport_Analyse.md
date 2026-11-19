# Rapport d'Analyse : Autonomous Mobile Robot ROS 2

## Résumé Exécutif
Ce document présente une analyse détaillée de l'architecture et du code du projet **autonomous-mobile-robot-ros2** (package `diff_robot`). Il met en évidence la structure modulaire, les choix techniques et l'intégration avancée de la stack de navigation (Nav2) et de simulation (Gazebo).

---

## 📁 1. Structure du Projet

Le projet est un package ROS 2 (Python) structuré de manière très standard et claire :

*   **`config/`** : Contient les fichiers de configuration, notamment `nav2_params.yaml` (paramètres détaillés pour la navigation autonome) et `navigation.rviz` (configuration de l'interface de visualisation 3D).
*   **`description/`** : Regroupe les fichiers de conception du robot au format URDF/Xacro. 
*   **`diff_robot/`** : Le code source Python du package, contenant notamment le script de téléopération (`teleop.py`).
*   **`launch/`** : Les scripts de lancement Python (`rsp_gazebo.launch.py` et `navigation.launch.py`) pour démarrer les différents nœuds ROS 2, la simulation et la stack de navigation.
*   **`maps/`** : Contient une carte 2D déjà générée de l'environnement (`my_map.yaml` et `my_map.pgm`), prête à être utilisée pour la navigation.
*   **`world/`** : Contient le monde de simulation 3D pour Gazebo (`diff_robot.world`).

---

## 💻 2. Architecture du Code

### Lancement et Simulation
*   **`launch/rsp_gazebo.launch.py`** : Ce script gère l'initialisation complète de la simulation. Il compile les fichiers `.xacro` en `.urdf` à la volée avec la bibliothèque `xacro`, lance le `robot_state_publisher`, démarre l'environnement Gazebo et y insère dynamiquement le robot simulé (`my_bot`).
*   **`launch/navigation.launch.py`** : Lance la pile de navigation complète (Nav2) en intégrant la carte pré-générée (`my_map.yaml`) et les configurations spécifiques (`nav2_params.yaml`), tout en ouvrant RViz2 pour la visualisation.

### Modélisation du Robot
*   **`description/control.xacro`** : Configure le plugin Gazebo `libgazebo_ros_diff_drive.so`. C'est le cœur du mouvement en simulation : il permet à Gazebo d'interpréter les commandes de vitesse (`/cmd_vel`) et de calculer l'odométrie (`/odom`) en fonction de l'écartement (0.35m) et du diamètre (0.1m) des roues.

### Contrôle et Interactions
*   **`teleop.py`** : Un nœud de contrôle manuel via le clavier très bien conçu. Il utilise `termios` et `tty` pour intercepter les frappes au clavier sans nécessiter d'appuyer sur "Entrée". Il permet de diriger le robot, mais aussi d'augmenter/diminuer dynamiquement la vitesse linéaire et angulaire avec des limites de sécurité strictes (`MAX_SPEED`, `MAX_TURN`).

---

## ⭐ 3. Points Forts de l'Implémentation

> [!TIP]
> **Architecture Modulaire (Xacro)**
> Au lieu d'avoir un fichier URDF monolithique, la conception du robot est séparée en plusieurs fichiers (`lidar.xacro`, `imu.xacro`, `inertial.xacro`, `control.xacro`). C'est une excellente pratique qui rend le code lisible, facile à maintenir et permet d'ajouter/retirer des capteurs très facilement.

> [!IMPORTANT]
> **Intégration avancée de Nav2**
> Le fichier `nav2_params.yaml` est extrêmement complet. On y voit un paramétrage fin du planificateur DWB (gestion des accélérations, décélérations, coûts de trajectoire) et des "costmaps" (couche d'obstacles, couche d'inflation). Cela démontre une volonté d'avoir une navigation fluide et sécurisée.

> [!NOTE]
> **Prêt à l'emploi (Out-of-the-box)**
> Le fait d'inclure une carte pré-générée (`my_map.pgm`) et un monde Gazebo personnalisé signifie que le projet peut être lancé et testé pour la navigation instantanément, sans avoir à faire d'étape de cartographie (SLAM) au préalable.

> [!NOTE]
> **Robustesse du script de Téléopération**
> Le script `teleop.py` gère le "clamping" (bridage des valeurs de vitesse) et intercepte les interruptions (Ctrl-C) pour envoyer une commande d'arrêt (0, 0) avant de quitter, évitant ainsi que le robot ne continue sur sa lancée si le script plante.

---

## 🔑 4. Points Clés du Projet

| Composant | Description |
| :--- | :--- |
| **Type de robot** | Robot à entraînement différentiel (Differential Drive) orienté vers la robotique mobile autonome. |
| **Capteurs simulés** | **LiDAR 2D** pour la détection d'obstacles et localisation (AMCL). Centrale inertielle **IMU** pour l'orientation précise. |
| **Stack technique** | **ROS 2** (Python / `ament_python`), **Gazebo** pour la simulation 3D physique, et **Nav2** pour l'intelligence de déplacement. |
