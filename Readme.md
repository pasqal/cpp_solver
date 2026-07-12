# CPP Solver

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](http://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![NetworkX 2.8+](https://img.shields.io/badge/networkx-2.8+-green.svg)](https://networkx.org/)
[![QGIS 3.0-4.x](https://img.shields.io/badge/qgis-3.0--4.x-brightgreen.svg)](https://qgis.org/)

> **Solves the Chinese Postman Problem (Route Inspection Problem)** \- Finds the shortest path (open or closed) that traverses every edge of a network at least once.

---

## 🌍 English Version

### 📚 Table of Contents

- [What is the CPP Solver Problem?](#what-is-the-chinese-postman-problem)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Input/Output Formats](#inputoutput-formats)
- [Algorithm Overview](#algorithm-overview)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

---

### 🤔 What is the Chinese Postman Problem?

The **Chinese Postman Problem (CPP)**, also known as the **Route Inspection Problem**, is a classic problem in graph theory. The goal is to find the **shortest path** (open or closed) that traverses **every edge of a graph at least once**. This is particularly useful for:

- **Route optimization** for delivery services, garbage collection, or street sweeping.
- **Network inspection** (e.g., checking power lines, pipelines).
- **Tour planning** where all roads must be visited.

If the start and end nodes are the same, the solution is a **closed circuit** (Eulerian circuit). Otherwise, it is an **open path** from start to end.

---

### ✨ Features

- **QGIS Plugin**: Seamless integration with QGIS 3.x and 4.x.
- **Interactive Node Selection**: Click on the map to select start and end nodes.
- **Automatic Segment Numbering**: All output segments are automatically numbered.
- **Open and Closed Paths**: Supports both open paths (start ≠ end) and closed circuits (start = end).
- **Customizable Style**: Output layer style can be customized via `cpp_solver.qml`.
- **Multiple Output Formats**: CSV, GPX (for GPS devices), and PNG (graph visualization).
- **Efficient Algorithm**: Uses the Blossom algorithm for optimal pair matching.
- **Handles Disconnected Graphs**: Automatically uses the largest connected component.
- **Open Source**: Licensed under MIT.

---

### 📋 Requirements

| Component | Version | Purpose |
|-----------|---------|---------|
| **QGIS** | 3.0 – 4.x | Core platform |
| **Python** | 3.8+ | Core runtime |
| **NetworkX** | 2.8+ | Graph algorithms |

---

### 💻 Installation

#### Method 1: Using QGIS Plugin Manager (Recommended)
1. Open QGIS.
2. Go to **Plugins → Manage and Install Plugins...**
3. Search for **"CPP Solver"**.
4. Click **Install Plugin**.
5. The plugin will be available under **Plugins → CPP Solver**.

#### Method 2: Manual Installation from Source
1. Clone this repository:
   ```bash
   git clone https://github.com/pasqal/cpp_solver.git
   cd cpp_solver
   ```
2. Run the installation script:
   ```bash
   ./install.sh
   ```
   This copies the plugin to your QGIS plugins directory.

3. Restart QGIS.

#### Method 3: Bundled Installation
1. Run the bundling script to create a ZIP file:
   ```bash
   ./bundle.sh
   ```
2. In QGIS, go to **Plugins → Manage and Install Plugins... → Install from ZIP**.
3. Select the generated ZIP file (`cpp_solver_bundle.zip`).

---

### 🚀 Usage

#### Step 1: Prepare Your Data
- Load a **vector line layer** (e.g., roads, paths, or any network) into QGIS.
- Ensure the layer has a **valid Coordinate Reference System (CRS)** in meters (e.g., UTM) for accurate distance calculations.

#### Step 2: Select Features
- Use the **"Select Features by Polygon"** tool to select the area of the network you want to analyze.
- Alternatively, select features manually.

#### Step 3: Run the Plugin
1. Go to **Plugins → CPP Solver → CPP Solver**.
2. The plugin will prompt you to:
   - Click on the map to select the **START node**.
   - Click again to select the **END node**.
3. The plugin will:
   - Compute the optimal CPP path.
   - Create a new **memory layer** named `chinese_postman` with the result.
   - Display a summary with:
     - Total length of roads (in km).
     - Total length of path (in km).
     - Length of sections visited twice (in km).
     - Number of segments.

#### Notes:
- If **START = END**, a **closed circuit** is generated.
- If **START ≠ END**, an **open path** from START to END is generated.
- Segments are **always numbered** in the output layer.

---

### 📂 Input/Output Formats

#### CSV Format

The plugin can export the result as a CSV file with the following columns:
- Start Node, End Node, Segment Length, Segment ID, Start Longitude, Start Latitude, End Longitude, End Latitude

#### GPX Output

The GPX file is a standard **GPS Exchange Format** file that can be imported into:
- GPS devices (Garmin, etc.).
- Mapping software (QGIS, Google Earth, etc.).

#### PNG Output

The PNG file is a **graph visualization** of the Eulerian path.
**Requirements:**
- [Graphviz](https://graphviz.org/) must be installed.
- One of the following Python packages: `pygraphviz`, `pydot`, or `pydotplus`.

---

### 🧮 Algorithm Overview

The plugin uses the following steps to solve the CPP:

1. **Identify Odd-Degree Nodes**: Find all nodes with an odd degree.
2. **Build Complete Graph of Odd Nodes**: Create a new graph where:
   - Nodes are the odd-degree nodes from the original graph.
   - Edges connect every pair of nodes.
   - Edge weights are the **shortest path distance** between the nodes in the original graph.
3. **Find Optimal Matching**: Use the **Blossom algorithm** (`nx.max_weight_matching`) to find the best pairing of odd nodes.
4. **Duplicate Edges**: For each matched pair (except start-end), duplicate the shortest path between them in the original graph.
5. **Find Eulerian Path/Circuit**: The modified graph now has exactly 0 or 2 odd-degree nodes, so an Eulerian path/circuit can be found.

**Complexity:**
- If V' (number of odd-degree nodes) is at least 10% of V (total nodes), the complexity is **O(V³)**.

---

### ⚠️ Troubleshooting

#### Common Issues

##### 1. Plugin Not Appearing
- **Cause**: Plugin not installed correctly or QGIS not restarted.
- **Solution**:
  - Restart QGIS.
  - Check that the plugin is in the correct directory (e.g., `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/`).
  - Run `./install.sh` again.

##### 2. Incorrect Distance Calculations
- **Cause**: The layer's **CRS is not in meters** (e.g., WGS84).
- **Solution**:
  - Reproject the layer to a **projected CRS** (e.g., UTM).
  - In QGIS: Right-click the layer → **Export → Save Features As...** → Choose a UTM zone.

##### 3. Plugin Crashes on Large Networks
- **Cause**: The algorithm has **O(V³)** complexity for large graphs.
- **Solution**:
  - Select a smaller subset of the network.
  - Use a more powerful computer.

##### 4. "Graph data not available" Error
- **Cause**: The plugin was restarted without reloading the layer.
- **Solution**:
  - Restart the plugin by clicking the CPP Solver button again.
  - Ensure you have selected valid line features.

##### 5. PNG Export Fails
- **Cause**: Graphviz or Python bindings not installed.
- **Solution**:
  - Install Graphviz: `sudo apt-get install graphviz` (Linux) or download from [graphviz.org](https://graphviz.org/).
  - Install Python bindings: `pip install pygraphviz` (recommended) or `pip install pydot`.

---

### 🤝 Contributing

Contributions are welcome! Here’s how you can help:

1. **Report Bugs**: Open an issue on [GitHub](https://github.com/pasqal/cpp_solver/issues).
2. **Suggest Features**: Open an issue with your feature request.
3. **Submit Code**:
   - Fork the repository.
   - Create a feature branch (`git checkout -b feature/your-feature`).
   - Commit your changes (`git commit -m 'Add some feature'`).
   - Push to the branch (`git push origin feature/your-feature`).
   - Open a Pull Request.

### Development Guidelines
- Follow **PEP 8** for Python code.
- Add **docstrings** to new functions.
- Include **unit tests** for new features.
- Update the **Readme.md** if you change user-facing functionality.

---

### 📜 License

This project is licensed under the **MIT License** – see the [LICENSE](LICENSE) file for details.

---

---

# Solveur CPP

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](http://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![NetworkX 2.8+](https://img.shields.io/badge/networkx-2.8+-green.svg)](https://networkx.org/)
[![QGIS 3.0-4.x](https://img.shields.io/badge/qgis-3.0--4.x-brightgreen.svg)](https://qgis.org/)

> **Résout le problème du postier chinois (Route Inspection Problem)** \- Trouve le plus court chemin (ouvert ou fermé) qui parcourt chaque arête d'un réseau au moins une fois.

---

## 📚 Table des matières

- [Qu'est-ce que le problème du CPP Solver ?](#quest-ce-que-le-problème-du-cpp-solver)
- [Fonctionnalités](#fonctionnalités)
- [Prérequis](#prérequis)
- [Installation](#installation-1)
- [Utilisation](#utilisation)
- [Formats d'entrée/sortie](#formats-dentrée-sortie)
- [Aperçu de l'algorithme](#aperçu-de-lalgorithme)
- [Résolution des problèmes](#résolution-des-problèmes)
- [Contribution](#contribution)
- [Licence](#licence-1)

---

### 🤔 Qu'est-ce que le problème du CPP Solver ?

Le **problème du postier chinois (CPP)**, également appelé **problème d'inspection de route**, est un problème classique en théorie des graphes. L'objectif est de trouver le **plus court chemin** (ouvert ou fermé) qui parcourt **chaque arête d'un graphe au moins une fois**. Cela est particulièrement utile pour :

- **L'optimisation des itinéraires** pour les services de livraison, la collecte des déchets ou le balayage des rues.
- **L'inspection de réseaux** (par exemple, vérification des lignes électriques, des pipelines).
- **La planification de tournées** où toutes les routes doivent être visitées.

Si les nœuds de départ et d'arrivée sont identiques, la solution est un **circuit fermé** (circuit eulérien). Sinon, il s'agit d'un **chemin ouvert** du départ à l'arrivée.

---

### ✨ Fonctionnalités

- **Plugin QGIS** : Intégration transparente avec QGIS 3.x et 4.x.
- **Sélection interactive des nœuds** : Cliquez sur la carte pour sélectionner les nœuds de départ et d'arrivée.
- **Numérotation automatique des segments** : Tous les segments de sortie sont automatiquement numérotés.
- **Chemins ouverts et fermés** : Prend en charge les chemins ouverts (départ ≠ arrivée) et les circuits fermés (départ = arrivée).
- **Style personnalisable** : Le style de la couche de résultat peut être personnalisé via `cpp_solver.qml`.
- **Plusieurs formats de sortie** : CSV, GPX (pour les appareils GPS) et PNG (visualisation du graphe).
- **Algorithme efficace** : Utilise l'algorithme Blossom pour l'appariement optimal des nœuds.
- **Gestion des graphes non connectés** : Utilise automatiquement la plus grande composante connectée.
- **Open Source** : Sous licence MIT.

---

### 📋 Prérequis

| Composant | Version | Usage |
|-----------|---------|-------|
| **QGIS** | 3.0 – 4.x | Plateforme principale |
| **Python** | 3.8+ | Environnement d'exécution |
| **NetworkX** | 2.8+ | Algorithmes de graphe |

---

### 💻 Installation

#### Méthode 1 : Via le gestionnaire de plugins QGIS (recommandé)
1. Ouvrez QGIS.
2. Allez dans **Plugins → Gérer et installer les plugins...**
3. Recherchez **"CPP Solver"**.
4. Cliquez sur **Installer le plugin**.
5. Le plugin sera disponible sous **Plugins → CPP Solver**.

#### Méthode 2 : Installation manuelle à partir du code source
1. Clonez ce dépôt :
   ```bash
   git clone https://github.com/pasqal/cpp_solver.git
   cd cpp_solver
   ```
2. Exécutez le script d'installation :
   ```bash
   ./install.sh
   ```
   Cela copie le plugin dans votre répertoire de plugins QGIS.

3. Redémarrez QGIS.

#### Méthode 3 : Installation via un bundle
1. Exécutez le script de bundling pour créer un fichier ZIP :
   ```bash
   ./bundle.sh
   ```
2. Dans QGIS, allez dans **Plugins → Gérer et installer les plugins... → Installer depuis un ZIP**.
3. Sélectionnez le fichier ZIP généré (`cpp_solver_bundle.zip`).

---

### 🚀 Utilisation

#### Étape 1 : Préparer vos données
- Chargez une **couche de lignes vectorielles** (par exemple, routes, chemins, ou tout réseau) dans QGIS.
- Assurez-vous que la couche a un **système de coordonnées de référence (SCR) valide** en mètres (par exemple, UTM) pour des calculs de distance précis.

#### Étape 2 : Sélectionner les entités
- Utilisez l'outil **"Sélectionner des entités par polygone"** pour sélectionner la zone du réseau que vous souhaitez analyser.
- Ou sélectionnez manuellement les entités.

#### Étape 3 : Exécuter le plugin
1. Allez dans **Plugins → CPP Solver → CPP Solver**.
2. Le plugin vous demandera de :
   - Cliquer sur la carte pour sélectionner le **nœud de départ**.
   - Cliquer à nouveau pour sélectionner le **nœud d'arrivée**.
3. Le plugin va :
   - Calculer le chemin CPP optimal.
   - Créer une nouvelle **couche mémoire** nommée `chinese_postman` avec le résultat.
   - Afficher un résumé avec :
     - Longueur totale des routes (en km).
     - Longueur totale du chemin (en km).
     - Longueur des sections visitées deux fois (en km).
     - Nombre de segments.

#### Remarques :
- Si **DÉPART = ARRIVÉE**, un **circuit fermé** est généré.
- Si **DÉPART ≠ ARRIVÉE**, un **chemin ouvert** de DÉPART à ARRIVÉE est généré.
- Les segments sont **toujours numérotés** dans la couche de résultat.

---

### 📂 Formats d'entrée/sortie

#### Format CSV

Le plugin peut exporter le résultat sous forme de fichier CSV avec les colonnes suivantes :
- Start Node, End Node, Segment Length, Segment ID, Start Longitude, Start Latitude, End Longitude, End Latitude

#### Sortie GPX

Le fichier GPX est un format standard **GPS Exchange Format** qui peut être importé dans :
- Les appareils GPS (Garmin, etc.).
- Les logiciels de cartographie (QGIS, Google Earth, etc.).

#### Sortie PNG

Le fichier PNG est une **visualisation du graphe** du chemin eulérien.
**Prérequis :**
- [Graphviz](https://graphviz.org/) doit être installé.
- L'un des packages Python suivants : `pygraphviz`, `pydot` ou `pydotplus`.

---

### 🧮 Aperçu de l'algorithme

Le plugin utilise les étapes suivantes pour résoudre le CPP :

1. **Identifier les nœuds de degré impair** : Trouver tous les nœuds avec un degré impair.
2. **Construire un graphe complet des nœuds impairs** : Créer un nouveau graphe où :
   - Les nœuds sont les nœuds de degré impair du graphe original.
   - Les arêtes relient chaque paire de nœuds.
   - Les poids des arêtes sont les **distances de plus court chemin** entre les nœuds dans le graphe original.
3. **Trouver l'appariement optimal** : Utiliser l'**algorithme Blossom** (`nx.max_weight_matching`) pour trouver le meilleur appariement des nœuds impairs.
4. **Dupliquer les arêtes** : Pour chaque paire appariée (sauf départ-arrivée), dupliquer le plus court chemin entre eux dans le graphe original.
5. **Trouver le chemin/circuit eulérien** : Le graphe modifié a maintenant exactement 0 ou 2 nœuds de degré impair, donc un chemin/circuit eulérien peut être trouvé.

**Complexité :**
- Si V' (nombre de nœuds de degré impair) représente au moins 10 % de V (nombre total de nœuds), la complexité est **O(V³)**.

---

### ⚠️ Résolution des problèmes

#### Problèmes courants

##### 1. Plugin non visible
- **Cause** : Plugin non installé correctement ou QGIS non redémarré.
- **Solution** :
  - Redémarrez QGIS.
  - Vérifiez que le plugin est dans le bon répertoire (par exemple, `~/.local/share/QGIS/QGIS3/profiles/default/python/plugins/`).
  - Exécutez à nouveau `./install.sh`.

##### 2. Calculs de distance incorrects
- **Cause** : Le **SCR de la couche n'est pas en mètres** (par exemple, WGS84).
- **Solution** :
  - Reprojetez la couche vers un **SCR projeté** (par exemple, UTM).
  - Dans QGIS : Cliquez droit sur la couche → **Exporter → Enregistrer les entités sous...** → Choisissez une zone UTM.

##### 3. Plantage du plugin sur les grands réseaux
- **Cause** : L'algorithme a une **complexité O(V³)** pour les grands graphes.
- **Solution** :
  - Sélectionnez un sous-ensemble plus petit du réseau.
  - Utilisez un ordinateur plus puissant.

##### 4. Erreur "Graph data not available"
- **Cause** : Le plugin a été relancé sans recharger la couche.
- **Solution** :
  - Relancez le plugin en cliquant à nouveau sur le bouton CPP Solver.
  - Assurez-vous d'avoir sélectionné des entités de type ligne valides.

##### 5. Échec de l'export PNG
- **Cause** : Graphviz ou les bindings Python ne sont pas installés.
- **Solution** :
  - Installez Graphviz : `sudo apt-get install graphviz` (Linux) ou téléchargez depuis [graphviz.org](https://graphviz.org/).
  - Installez les bindings Python : `pip install pygraphviz` (recommandé) ou `pip install pydot`.

---

### 🤝 Contribution

Les contributions sont les bienvenues ! Voici comment vous pouvez aider :

1. **Signaler des bugs** : Ouvrez une issue sur [GitHub](https://github.com/pasqal/cpp_solver/issues).
2. **Proposer des fonctionnalités** : Ouvrez une issue avec votre demande de fonctionnalité.
3. **Soumettre du code** :
   - Forkez le dépôt.
   - Créez une branche de fonctionnalité (`git checkout -b feature/ma-fonctionnalité`).
   - Validez vos modifications (`git commit -m 'Ajout d'une fonctionnalité'`).
   - Poussez vers la branche (`git push origin feature/ma-fonctionnalité`).
   - Ouvrez une Pull Request.

#### Directives de développement
- Respectez **PEP 8** pour le code Python.
- Ajoutez des **docstrings** aux nouvelles fonctions.
- Incluez des **tests unitaires** pour les nouvelles fonctionnalités.
- Mettez à jour le **Readme.md** si vous modifiez des fonctionnalités visibles par l'utilisateur.

---

### 📜 Licence

Ce projet est sous licence **MIT** – voir le fichier [LICENSE](LICENSE) pour plus de détails.
