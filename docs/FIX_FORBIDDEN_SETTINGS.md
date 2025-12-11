# 🔧 Fix: Paramètres d'Application Interdits - Solution Complète

## 🎯 Problème Résolu

Azure ajoutait automatiquement les paramètres interdits suivants :
- ❌ `AzureWebJobsStorage`
- ❌ `FUNCTIONS_WORKER_RUNTIME`

Ces paramètres bloquaient tous les déploiements sur Azure Static Web Apps.

## ✅ Solution Implémentée

### 1. Modification du Code (Commit 333e7c5)

**Changement effectué** : Suppression de `extensionBundle` dans `api/api/host.json`

**Avant :**
```json
{
  "version": "2.0",
  "logging": { ... },
  "extensionBundle": {
    "id": "Microsoft.Azure.Functions.ExtensionBundle",
    "version": "[4.*, 5.0.0)"
  }
}
```

**Après :**
```json
{
  "version": "2.0",
  "logging": { ... }
}
```

### 2. Pourquoi Cette Modification ?

L'`extensionBundle` indiquait à Azure que les fonctions devaient être traitées comme des **Azure Functions autonomes** (standalone), ce qui déclenchait l'ajout automatique des paramètres interdits.

En supprimant `extensionBundle`, Azure traite maintenant les fonctions comme des **fonctions gérées** (managed functions) pour Static Web Apps, ce qui est correct pour ce projet.

### 3. Est-ce Sûr ?

✅ **OUI** - Cette modification est sûre car :
- Toutes les fonctions utilisent uniquement des triggers HTTP (built-in)
- Aucune fonction n'utilise de bindings avancés (Queue, Blob, Table, etc.)
- L'`extensionBundle` n'est nécessaire que pour les bindings avancés
- Azure Static Web Apps gère automatiquement les bindings HTTP

## 📋 Actions Requises de Votre Côté

### Étape 1 : Supprimer les Paramètres Existants sur Azure

Les paramètres déjà présents dans Azure doivent être supprimés manuellement **une seule fois** :

#### Option A : Via Azure Portal (Recommandé)

1. Ouvrir https://portal.azure.com
2. Naviguer vers votre Static Web App
3. Menu gauche → **"Configuration"** → **"Application settings"**
4. Supprimer ces paramètres s'ils existent :
   - ❌ `AzureWebJobsStorage`
   - ❌ `FUNCTIONS_WORKER_RUNTIME`
   - ❌ `WEBSITE_NODE_DEFAULT_VERSION` (si présent)
5. Cliquer sur **"Save"**

#### Option B : Via Azure CLI (Script Automatique)

Le repository inclut un script de nettoyage automatique :

```bash
# Exécuter le script de nettoyage
cd scripts/
./fix-azure-settings.sh
```

Ce script supprime automatiquement tous les paramètres interdits via Azure CLI.

### Étape 2 : Déployer la Nouvelle Version

Une fois les paramètres supprimés dans Azure, déployez cette nouvelle version :

```bash
# Les commits sont déjà dans la branche
git checkout copilot/remove-problematic-application-settings
git push origin copilot/remove-problematic-application-settings

# Ou fusionner dans main
git checkout main
git merge copilot/remove-problematic-application-settings
git push origin main
```

### Étape 3 : Vérification

Après le déploiement (attendre 2-3 minutes) :

1. Vérifier qu'aucun paramètre interdit n'a été ajouté :
   - Azure Portal → Static Web App → Configuration → Application settings
   - Seuls les paramètres autorisés doivent être présents (AZURE_AI_API_KEY, etc.)

2. Vérifier que l'application fonctionne :
   ```bash
   # Remplacez <votre-app> par le nom de votre Static Web App
   curl https://<votre-app>.azurestaticapps.net/api/agents/axilum/invoke
   ```

## 🎉 Résultat Attendu

Après ces modifications :
- ✅ Azure ne devrait plus ajouter automatiquement les paramètres interdits
- ✅ Les nouveaux déploiements devraient fonctionner sans blocage
- ✅ L'application fonctionnera exactement comme avant

## ⚠️ Note Importante

Si Azure ajoute encore ces paramètres après cette modification, cela pourrait indiquer un problème de configuration au niveau de l'application Azure elle-même. Dans ce cas, contactez le support Azure.

## 📚 Références

- Documentation Azure : [Azure Static Web Apps Managed Functions](https://learn.microsoft.com/azure/static-web-apps/apis-functions)
- Différence entre managed et standalone functions
- Liste des paramètres interdits pour Static Web Apps
