# Installer GNU nano 📝 sur macOS Tahoe (bêta) sur architecture Intel

Ce guide détaille l'installation de l'éditeur de texte **GNU nano** (v8.5 ou plus récent) sur un Mac équipé de **macOS Tahoe (bêta 1)**. Il couvre la méthode recommandée via Homebrew et la compilation manuelle pour les utilisateurs avancés.

---

### ⚠️ Avertissements importants

- **Système en Bêta :** macOS Tahoe est une version de développement. Des instabilités ou des incompatibilités avec certains outils sont possibles. Procédez avec prudence et **sauvegardez vos données**.
- **Architecture :** Les commandes et chemins peuvent différer entre les Mac **Intel** (`/usr/local`) et **Apple Silicon** (`/opt/homebrew`). Ce guide mentionne les deux lorsque c'est pertinent.

---

## Table des matières

- [Prérequis](#prérequis)
- [Méthode 1 : Installation via Homebrew (Recommandée)](#méthode-1--installation-via-homebrew-recommandée)
- [Méthode 2 : Compilation manuelle depuis les sources](#méthode-2--compilation-manuelle-depuis-les-sources)
- [Étapes post-installation](#étapes-post-installation)
- [Dépannage](#dépannage)
- [Résumé des commandes](#résumé-des-commandes)
- [Ressources utiles](#ressources-utiles)

---

## Prérequis

Avant de commencer, assurez-vous d'avoir :

1.  **Xcode Command Line Tools (CLT) :** Installez la version bêta correspondant à macOS Tahoe depuis le [portail développeur Apple](https://developer.apple.com/download/all/) ou via la commande :
    ```sh
    xcode-select --install
    ```

2.  **Homebrew :** Le gestionnaire de paquets pour macOS. S'il n'est pas installé, exécutez :
    ```sh
    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    ```

---

## Méthode 1 : Installation via Homebrew (Recommandée)

C'est la méthode la plus simple et la plus fiable pour la majorité des utilisateurs.

### Étapes

1.  **Configurer le PATH de Homebrew**
    Assurez-vous que votre shell peut trouver les commandes installées par Homebrew.
    ```sh
    # Pour Mac Intel
    echo 'eval "$(/usr/local/bin/brew shellenv)"' >> ~/.zshrc

    # Pour Mac Apple Silicon
    echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
    
    # Appliquez les changements
    source ~/.zshrc
    ```
    > Remplacez `~/.zshrc` par `~/.bash_profile` si vous utilisez Bash.

2.  **Diagnostiquer Homebrew**
    Vérifiez que votre installation de Homebrew est saine.
    ```sh
    brew doctor
    ```
    > Suivez les recommandations pour corriger les éventuels problèmes.

3.  **Installer nano**
    Mettez à jour les formules Homebrew et installez nano.
    ```sh
    brew update
    brew install nano
    ```

Passez ensuite à la section [Étapes post-installation](#étapes-post-installation) pour vérifier et configurer nano.

---

## Méthode 2 : Compilation manuelle depuis les sources

Cette méthode avancée offre un contrôle total sur les options de compilation. Elle est utile si Homebrew échoue ou pour des besoins spécifiques.

### Étapes

1.  **Installer les dépendances**
    Utilisez Homebrew pour installer les bibliothèques requises.
    ```sh
    brew install ncurses gettext libmagic
    ```
    - `ncurses` : pour l'interface en mode texte.
    - `gettext` : pour la gestion des traductions (internationalisation).
    - `libmagic` : pour la détection automatique du type de fichier (coloration syntaxique).

2.  **Télécharger et extraire les sources de nano**
    ```sh
    cd ~/Downloads
    curl -O https://www.nano-editor.org/dist/v8/nano-8.5.tar.xz
    tar -xf nano-8.5.tar.xz
    cd nano-8.5
    ```

3.  **Configurer le script de compilation**
    Le script `./configure` prépare les fichiers pour la compilation.
    ```sh
    ./configure --enable-utf8 --enable-libmagic
    ```
    > **💡 Note sur les dépendances non trouvées :**
    > Si le script se plaint de ne pas trouver `ncurses` ou une autre bibliothèque, vous devrez spécifier manuellement les chemins d'accès de Homebrew.
    > ```sh
    > # Exécutez ceci AVANT ./configure
    > export LDFLAGS="-L$(brew --prefix ncurses)/lib -L$(brew --prefix gettext)/lib -L$(brew --prefix libmagic)/lib"
    > export CPPFLAGS="-I$(brew --prefix ncurses)/include -I$(brew --prefix gettext)/include -I$(brew --prefix libmagic)/include"
    > 
    > # Relancez ensuite la configuration
    > ./configure --enable-utf8 --enable-libmagic
    > ```
    > La commande `brew --prefix <formule>` retourne le chemin d'installation correct, que ce soit sur Intel ou Apple Silicon.

4.  **Compiler et installer**
    ```sh
    make      # Compile le code source
    make check  # (Optionnel mais recommandé) Lance la suite de tests
    sudo make install # Installe nano dans /usr/local/bin
    ```

Passez ensuite à la section suivante pour finaliser l'installation.

---

## Étapes post-installation

### ✅ Vérification
Quelle que soit la méthode utilisée, vérifiez que la version installée est la bonne et qu'elle est prioritaire sur celle du système.

```sh
which nano
```

> Doit retourner :

>  * `/opt/homebrew/bin/nano` (Apple Silicon)
>  * `/usr/local/bin/nano` (Intel)
>  * ... et non `/usr/bin/nano` (la version système).

> 
```sh
nano --version
```

> Doit afficher `GNU nano, version 8.5` ou une version plus récente.
> 
### 🎨 Configuration de la coloration syntaxique
Pour activer les définitions de coloration syntaxique par défaut (y compris celles qui utilisent libmagic), éditez ou créez le fichier `~/.nanorc` :

Cela crée le fichier s'il n'existe pas et l'ouvre
`nano ~/.nanorc`

Ajoutez cette ligne à l'intérieur :
```text
include "$(brew --prefix)/share/nano/*.nanorc"
```

> Utiliser `$(brew --prefix)` garantit que le chemin est correct sur toutes les architectures.
> 
### Dépannage
 * Problème de *PATH* : Si la commande which nano retourne `/usr/bin/nano`, votre *PATH* n'est pas correctement configuré. Revoyez l'étape 1 de la méthode Homebrew.
 * Erreurs de compilation : Lisez attentivement les erreurs. Le plus souvent, il s'agit d'une dépendance manquante ou d'un chemin mal configuré (**LDFLAGS**/**CPPFLAGS**).
 * Conflits avec la version système : Assurez-vous que `/usr/local/bin` (Intel) ou `/opt/homebrew/bin` (Apple Silicon) est listé avant `/usr/bin` dans votre variable d'environnement **$PATH**.
 * Problèmes liés à la bêta : Consultez les discussions sur le dépôt GitHub de Homebrew pour des problèmes spécifiques à macOS Tahoe.

### Résumé des commandes
| Action | Commande |
|---|---|
| Installer/Mettre à jour les CLT de Xcode | `xcode-select --install` |
| Installer Homebrew | `/bin/bash -c "$(curl ...)"` |
| Configurer le PATH pour Homebrew (Zsh) | `echo 'eval "$($(brew --prefix)/bin/brew shellenv)"' >> ~/.zshrc && source ~/.zshrc` |
| Diagnostiquer Homebrew | `brew doctor` |
| Installer via Homebrew | `brew update && brew install nano` |
| Installer dépendances (manuel) | `brew install ncurses gettext libmagic` |
| Configurer (manuel) | `./configure --enable-utf8 --enable-libmagic` |
| Compiler et installer (manuel) | `make && make check && sudo make install` |
| Vérifier le chemin d'installation | `which nano` |
| Vérifier la version | `nano --version` |

### Ressources utiles
 * Site officiel de GNU nano
 * Site officiel de Homebrew
 * Page de la formule nano sur Homebrew
 * Téléchargements pour développeurs Apple.
   
Bonne édition avec nano sur macOS Tahoe !
