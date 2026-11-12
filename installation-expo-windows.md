# Installation de React Native (Expo) sur Windows, de A à Z

Guide testé sur Windows 11 avec VS Code, Android Studio et un émulateur Android.

---

## 1. Prérequis à installer

| Outil | Rôle |
|---|---|
| Node.js (LTS) + npm | Exécute Expo et Metro |
| JDK 17 | Requis par les outils Android |
| Android Studio | Fournit le SDK Android et l'émulateur |
| VS Code | Éditeur de code |

### Installation rapide avec winget (PowerShell)

```powershell
winget install OpenJS.NodeJS.LTS
winget install Microsoft.OpenJDK.17
winget install Microsoft.VisualStudioCode
```

Android Studio se télécharge sur : https://developer.android.com/studio

> Après chaque installation, **ferme et rouvre le terminal**.

### Vérifier Node et npm

```powershell
node -v
npm -v
```

> Attention : c'est `node -v` (un seul tiret). `node -version` donne une erreur.

---

## 2. Configurer Android Studio

1. Ouvre Android Studio, puis **More Actions > SDK Manager**.
2. Onglet **SDK Platforms** : installe une version récente d'Android.
3. Onglet **SDK Tools** : vérifie que ces éléments sont installés :
   - Android SDK Build-Tools
   - Android SDK Platform-Tools
   - Android SDK Command-line Tools
   - Android Emulator
4. Clique sur **Apply**.

---

## 3. Variables d'environnement

Remplace `TON_NOM` par ton nom d'utilisateur Windows si besoin.

### Trouver le chemin exact du JDK

```powershell
Get-ChildItem "C:\Program Files\Microsoft"
```

Exemple de résultat : `jdk-17.0.21.21-hotspot`

### Définir les variables (PowerShell)

```powershell
[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Program Files\Microsoft\jdk-17.0.21.21-hotspot", "User")
[Environment]::SetEnvironmentVariable("ANDROID_HOME", "$env:LOCALAPPDATA\Android\Sdk", "User")
```

### Ajouter au Path (à exécuter UNE SEULE FOIS)

```powershell
$p = [Environment]::GetEnvironmentVariable("Path", "User")
$add = "$env:LOCALAPPDATA\Android\Sdk\platform-tools;$env:LOCALAPPDATA\Android\Sdk\emulator"
[Environment]::SetEnvironmentVariable("Path", "$p;$add", "User")
```

Si `java` n'est pas reconnu après redémarrage du terminal, ajoute aussi son dossier `bin` (une seule fois) :

```powershell
$p = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$p;C:\Program Files\Microsoft\jdk-17.0.21.21-hotspot\bin", "User")
```

### Vérifier (dans un NOUVEAU terminal)

> Les variables ne s'appliquent qu'aux terminaux ouverts **après** leur création. Ferme tout (PowerShell, VS Code) puis rouvre.

```powershell
java -version
echo $env:JAVA_HOME
echo $env:ANDROID_HOME
adb --version
```

Résultats attendus :
- `java -version` affiche une version 17.x
- `adb --version` affiche « Android Debug Bridge version ... »

---

## 4. Créer et lancer l'émulateur

### Depuis Android Studio

**More Actions > Virtual Device Manager**, puis clique sur ▶ à côté de ton appareil.

### Depuis PowerShell

```powershell
emulator -list-avds
emulator -avd NOM_DE_TON_EMULATEUR
```

Exemple : `emulator -avd Pixel_9a`

Laisse ce terminal ouvert. Attends l'écran d'accueil d'Android (1 à 2 minutes au premier démarrage).

### Vérifier que l'émulateur est détecté

Dans un deuxième terminal :

```powershell
adb devices
```

Tu dois voir une ligne comme `emulator-5554   device`.

### Si l'émulateur ne démarre pas

- Active la virtualisation (VT-x ou SVM) dans le BIOS.
- Active **Plateforme de l'hyperviseur Windows** dans les fonctionnalités Windows, puis redémarre.
- Si l'écran est noir ou très lent : Device Manager > Edit > Graphics > **Hardware**.

---

## 5. Créer le projet Expo

```powershell
cd C:\Users\TON_NOM\Documents
npx create-expo-app@latest MonApp
cd MonApp
code .
```

---

## 6. Lancer l'app sur l'émulateur

Avec l'émulateur déjà démarré, dans le terminal de VS Code (`` Ctrl+` ``) :

```powershell
npx expo start
```

Puis appuie sur **`a`** dans ce terminal. Expo télécharge et installe **Expo Go** sur l'émulateur, puis ouvre ton projet.

Ou directement :

```powershell
npx expo start --android
```

> Au premier lancement, le téléchargement d'Expo Go et la compilation par Metro prennent du temps. C'est normal.

### Raccourcis utiles dans le terminal d'Expo

| Touche | Action |
|---|---|
| `a` | Ouvrir sur Android |
| `r` | Recharger l'app |
| `m` | Menu développeur |
| `Ctrl+C` | Arrêter le serveur |

---

## 7. Modifier l'app (exemple : « Salut Ali »)

Le fichier d'accueil dépend du projet. Si le terminal affiche `Using src/app as the root directory for Expo Router`, ouvre **`src/app/index.tsx`**. Sinon, il se trouve dans `app/index.tsx` ou `app/(tabs)/index.tsx`.

Remplace son contenu par :

```tsx
import { StyleSheet, Text, View } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Salut toi</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  text: {
    fontSize: 32,
    fontWeight: 'bold',
  },
});
```

Sauvegarde avec `Ctrl+S` : l'émulateur se met à jour tout seul (Fast Refresh). Sinon, appuie sur `r` dans le terminal.

---

## 8. Extensions VS Code recommandées

- ES7+ React/Redux/React-Native snippets
- ESLint
- Prettier
- Expo Tools
- React Native Tools (Microsoft)

---

## 9. Dépannage

| Problème | Solution |
|---|---|
| `java` ou `adb` non reconnu | Ferme et rouvre **tous** les terminaux. Vérifie `JAVA_HOME`, `ANDROID_HOME` et le `Path`. |
| `adb devices` ne montre rien | L'émulateur n'est pas démarré. Lance-le (étape 4). |
| `Failed to resolve the Android SDK path` | Vérifie `echo $env:ANDROID_HOME` et que le dossier existe. |
| `No Android connected device found` | L'émulateur n'est pas détecté par `adb`. Relance-le puis `adb devices`. |
| Expo Go ne s'ouvre pas | Ouvre Expo Go dans l'émulateur et choisis ton serveur, ou relance `npx expo start --android`. |
| Connexion impossible à Metro | `adb reverse tcp:8081 tcp:8081` |
| Réseau problématique | `npx expo start --tunnel` (accepte d'installer `@expo/ngrok`). |
| Plusieurs versions de Node (nvm-windows) | `nvm list`, `nvm install 22`, `nvm use 22` |

---

## 10. Notes

- iOS est impossible sur Windows (il faut un Mac et Xcode). Pour tester sur iPhone, utilise l'app **Expo Go** avec ton iPhone.
- Les numéros de version changent régulièrement. Vérifie la documentation officielle : https://docs.expo.dev et https://reactnative.dev/docs/set-up-your-environment
