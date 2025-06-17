# Problème de support UTF-8 lors de la compilation de nano sur macOS

## Erreur rencontrée

Lors de l’exécution de la commande `./configure` pour compiler nano, l’erreur
suivante peut apparaître :

```text
configure: error:
  *** UTF-8 support was requested, but insufficient support was
  *** detected in your curses and/or C libraries.  Please verify
  *** that both your curses library and your C library were built
  *** with wide-character support.
```

---

### Explication

- Cette erreur signifie que la bibliothèque `ncurses` utilisée pour la
  compilation n’est pas détectée avec le support des caractères larges
  (wide-character), indispensable pour le support UTF-8 dans nano(1)(4).
- Sur macOS, contrairement à Linux, il n’existe pas de bibliothèque séparée
  `ncursesw` : le support large est intégré dans la version principale de
  `ncurses`, mais la version système de macOS est souvent trop ancienne ou
  incomplète(1)(3)(4).

---

### Solution recommandée

1. **Installer ncurses via Homebrew**  
   Homebrew installe `ncurses` dans un chemin spécifique
   (`/usr/local/opt/ncurses` sur Intel), sans le symlinker dans `/usr/local` par
   défaut(3)(4)(7).
2. **Forcer l’utilisation de la version Homebrew**  
   Avant de relancer la configuration, exportez ces variables :

   ```sh
   export CPPFLAGS="-I/usr/local/opt/ncurses/include"
   export LDFLAGS="-L/usr/local/opt/ncurses/lib"
   export PKG_CONFIG_PATH="/usr/local/opt/ncurses/lib/pkgconfig"
   ```

   *(Adaptez `/usr/local` à `/opt/homebrew` sur Apple Silicon.)*

3. **Relancer la configuration**  
   Lancez :

   ```sh
   ./configure --enable-utf8 --enable-libmagic
   ```

4. **Compiler et installer**  

   ```sh
   make
   make check
   sudo make install
   ```

---

### Points d’attention

- N’utilisez pas `--with-magic` (option ignorée), mais bien
  `--enable-libmagic`(5).
- Vérifiez que c’est bien la version Homebrew de `ncurses` qui est utilisée si
  plusieurs versions existent(3)(4)(7).
- Redémarrez le terminal après modification des variables d’environnement si
  besoin(4).

---

### Résumé

- Le problème provient d’un conflit entre la version système de `ncurses`
  (incomplète) et celle de Homebrew(1)(3)(4)(7).
- Il faut explicitement indiquer au système de build d’utiliser la version
  Homebrew via `CPPFLAGS`, `LDFLAGS` et `PKG_CONFIG_PATH`(3)(4)(6)(7).
- Relancez la configuration et la compilation comme indiqué ci-dessus.

---

### Pour aller plus loin

- [ncurses Homebrew Formulae][https://formulae.brew.sh/formula/ncurses](7)
- [Explications sur le support wide-character sous macOS][https://stackoverflow.com/questions/48042203/curses-library-doesnt-support-wide-char-on-os-x-high-sierra](1)
- [Configurer les variables d’environnement pour Homebrew ncurses][https://apple.stackexchange.com/questions/211497/install-new-nano-2-4-using-brew-but-still-uses-old-versions-symbolic-link-not](4)
- [Reddit : gestion des chemins Homebrew et ncurses sur macOS][https://www.reddit.com/r/zsh/comments/kjgxws/did_zsh_install_correctly/](3)
- [How do I know where homebrew installs symlinks? - Stack Overflow][https://stackoverflow.com/questions/30675064/how-do-i-know-where-homebrew-installs-symlinks](6)
- [Compile nano on macOS - Stack Overflow][https://stackoverflow.com/questions/79318615/compile-nano-on-macos](5)

---

*En suivant ces étapes, la compilation de nano avec le support UTF-8 devrait
fonctionner correctement sur macOS Sequoia [Intel](1)(3)(4)(6)(7).*
