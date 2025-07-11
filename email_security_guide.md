# Guide complet : Sécurisation email d'un domaine (Niveau Expert)

*Procédure complète appliquée sur lemieuxsamson.com - Réplicable sur n'importe quel domaine*

## 📋 Prérequis

- ✅ Domaine actif avec accès DNS
- ✅ Service email configuré (Microsoft 365, Google Workspace, etc.)
- ✅ Accès au panneau de contrôle DNS du domaine
- ✅ Compte GitHub (pour MTA-STS/BIMI)

---

## 🛡️ ÉTAPE 1 : Configuration SPF (Sender Policy Framework)

**Objectif :** Spécifier quels serveurs sont autorisés à envoyer des emails pour votre domaine.

### 1.1 Identifier votre fournisseur email
- **Microsoft 365 :** `spf.protection.outlook.com`
- **Google Workspace :** `_spf.google.com`  
- **Autre :** Consulter la documentation du fournisseur

### 1.2 Créer l'enregistrement SPF
```dns
@ TXT "v=spf1 include:spf.protection.outlook.com -all"
```

**Paramètres importants :**
- `v=spf1` : Version SPF
- `include:` : Inclut les serveurs du fournisseur
- `-all` : Rejette tout autre serveur (politique stricte)

### 1.3 Validation
- Tester sur [MXToolbox SPF Check](https://mxtoolbox.com/spf.aspx)
- Vérifier avec `dig TXT votre-domaine.com`

---

## 🔐 ÉTAPE 2 : Configuration DKIM (DomainKeys Identified Mail)

**Objectif :** Signer cryptographiquement vos emails pour prouver leur authenticité.

### 2.1 Activer DKIM chez votre fournisseur

**Microsoft 365 :**
1. Admin Center > Sécurité > Email et collaboration > Politiques et règles
2. Politiques de menace > DKIM
3. Sélectionner votre domaine > Activer

**Google Workspace :**
1. Admin Console > Applications > Google Workspace > Gmail
2. Authentifier les emails > Générer une nouvelle clé

### 2.2 Ajouter les enregistrements CNAME
Le fournisseur vous donnera des enregistrements comme :
```dns
selector1._domainkey CNAME selector1-votre-domaine-com._domainkey.provider.com
selector2._domainkey CNAME selector2-votre-domaine-com._domainkey.provider.com
```

### 2.3 Validation
- Attendre 15-30 minutes pour propagation
- Tester sur [MXToolbox DKIM](https://mxtoolbox.com/dkim.aspx)

---

## 🚫 ÉTAPE 3 : Configuration DMARC (Domain-based Message Authentication)

**Objectif :** Politique stricte contre l'usurpation d'emails.

### 3.1 Déploiement progressif

**Phase 1 - Monitoring (1-2 semaines) :**
```dns
_dmarc TXT "v=DMARC1; p=none; rua=mailto:dmarc@votre-domaine.com; fo=1"
```

**Phase 2 - Quarantaine (1-2 semaines) :**
```dns
_dmarc TXT "v=DMARC1; p=quarantine; rua=mailto:dmarc@votre-domaine.com; fo=1"
```

**Phase 3 - Production (configuration finale) :**
```dns
_dmarc TXT "v=DMARC1; p=reject; sp=reject; rua=mailto:admin@votre-domaine.com; fo=1"
```

### 3.2 Paramètres expliqués
- `p=reject` : Rejeter les emails non conformes
- `sp=reject` : Même politique pour les sous-domaines
- `rua=` : Adresse pour rapports agrégés
- `fo=1` : Rapports détaillés en cas d'échec

### 3.3 Validation
- Tester sur [DMARC Analyzer](https://www.dmarcanalyzer.com/)
- Surveiller les rapports reçus

---

## 🔒 ÉTAPE 4 : Configuration MTA-STS (Mail Transfer Agent Strict Transport Security)

**Objectif :** Forcer le chiffrement TLS pour tous les emails entrants.

### 4.1 Créer un repository GitHub

1. **Créer un nouveau repository** : `votre-domaine-security`
2. **Visibilité :** Public
3. **Initialiser** avec README

### 4.2 Structure des fichiers

**Créer ces fichiers dans le repository :**

#### `.nojekyll` (fichier vide)
*Nécessaire pour que GitHub Pages serve les dossiers `.well-known`*

#### `.well-known/mta-sts.txt`
```
version: STSv1
mode: enforce
mx: votre-domaine-com.mail.protection.outlook.com
max_age: 604800
```

#### `bimi/logo.svg` (optionnel pour BIMI)
```svg
<?xml version="1.0" encoding="UTF-8"?>
<svg version="1.2" baseProfile="tiny" xmlns="http://www.w3.org/2000/svg" 
     viewBox="0 0 512 512" width="512" height="512">
  <rect width="512" height="512" fill="#003366"/>
  <text x="256" y="280" text-anchor="middle" fill="white" 
        font-family="Arial" font-size="120" font-weight="bold">VD</text>
</svg>
```

#### `index.html` (page d'accueil)
```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Votre Domaine - Sécurité Email</title>
    <style>
        body { font-family: Arial; background: #003366; color: white; text-align: center; padding: 50px; }
        .feature { background: rgba(255,255,255,0.1); margin: 20px; padding: 20px; border-radius: 10px; }
        .status { background: #00CC66; padding: 5px 15px; border-radius: 20px; }
    </style>
</head>
<body>
    <h1>Votre Domaine</h1>
    <p>Services de sécurité email configurés</p>
    <div class="feature">
        <h3>MTA-STS</h3>
        <span class="status">✓ Actif</span>
    </div>
</body>
</html>
```

### 4.3 Activer GitHub Pages

1. **Repository Settings** > **Pages**
2. **Source :** Deploy from a branch
3. **Branch :** main / (root)
4. **Custom domain :** `security.votre-domaine.com`
5. **Enforce HTTPS :** ✓

### 4.4 Configuration DNS

```dns
; MTA-STS
_mta-sts TXT "v=STSv1; id=20250711001"
security CNAME votre-username.github.io.

; TLS Reporting (optionnel)
_smtp._tls TXT "v=TLSRPTv1; rua=mailto:admin@votre-domaine.com"
```

### 4.5 Validation
- Tester : `https://security.votre-domaine.com/.well-known/mta-sts.txt`
- Vérifier sur [MXToolbox MTA-STS](https://mxtoolbox.com/mta-sts.aspx)

---

## 🎨 ÉTAPE 5 : Configuration BIMI (Brand Indicators for Message Identification)

**Objectif :** Afficher votre logo dans les clients email.

### 5.1 Prérequis BIMI
- ✅ DMARC en mode `p=reject` ou `p=quarantine`
- ✅ Logo au format SVG Tiny 1.2
- ✅ Hébergement HTTPS (déjà fait avec GitHub Pages)

### 5.2 Enregistrement DNS BIMI
```dns
default._bimi TXT "v=BIMI1; l=https://security.votre-domaine.com/bimi/logo.svg"
```

### 5.3 Validation
- Tester sur [BIMI Inspector](https://bimigroup.org/bimi-generator/)
- Envoyer un email test vers Gmail pour voir le logo

---

## 🛡️ ÉTAPE 6 : Configuration DNSSEC

**Objectif :** Protéger vos enregistrements DNS contre la manipulation.

### 6.1 Option A : Chez votre registrar actuel
1. **Panneau de contrôle** du registrar
2. **Chercher "DNSSEC"** dans les paramètres
3. **Activer DNSSEC** (génération automatique des clés)

### 6.2 Option B : Migration vers Cloudflare (recommandé)
1. **Créer compte Cloudflare** (gratuit)
2. **Ajouter votre domaine**
3. **Importer les enregistrements DNS** existants
4. **Changer les nameservers** chez le registrar
5. **Activer DNSSEC** en 1 clic dans Cloudflare
6. **Configurer DS Records** chez le registrar avec les valeurs Cloudflare

### 6.3 Validation
- Tester sur [DNSViz](https://dnsviz.net/)
- Vérifier avec `dig +dnssec votre-domaine.com`

---

## 📊 CONFIGURATION DNS FINALE

Voici l'exemple complet des enregistrements DNS :

```dns
; SPF - Authentication
@ TXT "v=spf1 include:spf.protection.outlook.com -all"

; DKIM - Signatures (générés par le fournisseur)
selector1._domainkey CNAME selector1-votre-domaine-com._domainkey.provider.com
selector2._domainkey CNAME selector2-votre-domaine-com._domainkey.provider.com

; DMARC - Politique anti-spoofing
_dmarc TXT "v=DMARC1; p=reject; sp=reject; rua=mailto:admin@votre-domaine.com; fo=1"

; MTA-STS - Transport sécurisé
_mta-sts TXT "v=STSv1; id=20250711001"
security CNAME votre-username.github.io.

; BIMI - Logo de marque
default._bimi TXT "v=BIMI1; l=https://security.votre-domaine.com/bimi/logo.svg"

; TLS Reporting - Monitoring
_smtp._tls TXT "v=TLSRPTv1; rua=mailto:admin@votre-domaine.com"

; Enregistrements de base (à adapter)
@ MX 0 votre-domaine-com.mail.protection.outlook.com.
@ A 198.16.146.197
www CNAME @
```

---

## ✅ VALIDATION COMPLÈTE

### Outils de test recommandés
1. **[MXToolbox](https://mxtoolbox.com/)** - Tests SPF, DKIM, DMARC, MTA-STS
2. **[DNSViz](https://dnsviz.net/)** - Validation DNSSEC
3. **[BIMI Inspector](https://bimigroup.org/bimi-generator/)** - Test BIMI
4. **[Hardenize](https://www.hardenize.com/)** - Audit complet de sécurité

### Test final par email
1. **Envoyer un email** depuis votre domaine vers Gmail
2. **Vérifier dans Gmail :**
   - Logo BIMI affiché ✓
   - Examiner les headers pour DMARC PASS ✓
   - Confirmer TLS forcé (MTA-STS) ✓

---

## 📈 MONITORING CONTINU

### Surveillance recommandée
- **Rapports DMARC** (quotidiens/hebdomadaires)
- **Rapports TLS** (détection des problèmes de chiffrement)
- **Status DNSSEC** (alertes en cas de problème)
- **Expiration certificats** SSL

### Services de monitoring
- **DMARC Analyzer** - Analyse automatisée des rapports
- **Mailhardener** - Surveillance email complète
- **Cloudflare Analytics** - Si vous utilisez Cloudflare

---

## 🎯 RÉSULTAT FINAL

**Score de sécurité email : A+** 🏆

Votre domaine bénéficie maintenant de :
- ✅ **Authentication complète** (SPF + DKIM + DMARC)
- ✅ **Transport sécurisé** (MTA-STS + TLS-RPT)
- ✅ **Protection DNS** (DNSSEC)
- ✅ **Branding professionnel** (BIMI)
- ✅ **Monitoring intégré**

**Temps total d'implémentation :** 2-4 heures  
**Coût :** Gratuit (sauf certificat VMC pour BIMI si souhaité)  
**Niveau de sécurité :** Expert - parmi les mieux protégés

---

## 📝 NOTES IMPORTANTES

### Ordre d'implémentation recommandé
1. **SPF** → 2. **DKIM** → 3. **DMARC (progressif)** → 4. **MTA-STS** → 5. **BIMI** → 6. **DNSSEC**

### Temps de propagation
- **DNS records** : 5-30 minutes
- **DNSSEC** : 1-24 heures
- **GitHub Pages** : 5-10 minutes

### Domaines sans service email
Pour des domaines parkés, utilisez :
```dns
@ TXT "v=spf1 -all"
_dmarc TXT "v=DMARC1; p=reject; sp=reject; rua=mailto:admin@votre-domaine-principal.com"
@ MX 0 .
```

### Future expansion
- **TLSA Records** : Quand vous ajouterez des services web
- **CAA Records** : Pour contrôler l'émission de certificats SSL
- **HSTS** : Pour forcer HTTPS sur le site web

---

*Guide créé le 11 juillet 2025 - Applicable à tout domaine avec adaptation des paramètres spécifiques*
