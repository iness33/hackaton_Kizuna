# Mieru (見える) — le pont Kizuna

Application web progressive (PWA) du hackathon Kizuna : elle lit le compteur électrique des familles tunisiennes et leur montre, avant la facture, à quelle distance elles sont du seuil des 200 kWh. Charte : estampe japonaise × Sidi Bou Saïd, mascotte **Aaddad** (le compteur vivant).

## Démarrer en 3 minutes

Prérequis : Node.js 18 ou plus récent.

```bash
npm install
npm run dev
```

Ouvre l'adresse affichée (par exemple `http://localhost:5173`). Pour tester sur ton téléphone, il doit être sur le même Wi-Fi : utilise l'adresse « Network » affichée par Vite.

> La caméra du navigateur ne fonctionne qu'en **HTTPS** ou sur `localhost`. Pour un HTTPS rapide : onglet **PORTS** de VS Code → Forward a Port `5173` → visibilité **Public** (déjà autorisé dans `vite.config.js`). Sur le téléphone via l'adresse réseau, le bouton « Choisir une photo » ouvre quand même l'appareil photo. Pour la démo, déploie sur Vercel (HTTPS automatique).

Sur l'écran de bienvenue, **« Essayer avec la démo Maison Ben Salah »** crée un compteur avec 3 membres et un mois de relevés (168 kWh consommés).

Tests du moteur de calcul : `npm test`.

## Deux modes de données

| Mode | Quand | Ce qui marche |
| --- | --- | --- |
| **Local** | Par défaut, aucune configuration | Tout, sur un seul téléphone (localStorage + IndexedDB). Le partage Kizuna reste limité à ce téléphone. |
| **Cloud** | `.env.local` rempli | Tout + le **partage Kizuna en temps réel** entre téléphones, photos sur Supabase et contribution photovoltaïque agrégée. |

### Activer Supabase (mode cloud, ~10 minutes)

1. Crée un projet gratuit sur supabase.com.
2. **SQL Editor → New query** : colle `supabase/schema.sql`, puis **Run**. Le script crée les tables, le bucket de photos et active le temps réel.
3. **Project Settings → API** : copie l'URL du projet et la clé `anon public`.
4. Copie `.env.example` en `.env.local` et remplis les deux valeurs.
5. Relance `npm run dev`. Les comptes créés en mode local ne sont pas transférés.

Les règles d'accès sont simplifiées pour le hackathon : quiconque a l'identifiant d'un compteur peut le lire et y écrire.

### Activer l'assistant IA Aaddad

L'assistant utilise OpenAI via une Supabase Edge Function. Les messages sont envoyés à OpenAI pour générer les réponses ; les relevés du compteur ne sont pas transmis et les conversations ne sont pas enregistrées. Une limite de 20 messages par jour et par adresse réseau est appliquée côté serveur ; seule une empreinte temporaire de l'adresse est conservée.

1. Dans Supabase **SQL Editor**, exécute `supabase/assistant.sql`.
2. Installe la [Supabase CLI](https://supabase.com/docs/guides/cli), lie le projet avec `supabase link --project-ref <project-ref>`, puis ajoute ta clé OpenAI comme secret serveur : `supabase secrets set "OPENAI_API_KEY=ta-cle"` (remplace `ta-cle` par ta vraie clé).
3. Depuis le dossier du projet, déploie la fonction : `supabase functions deploy aaddad-assistant`.
4. Configure `.env.local` avec l'URL Supabase et la clé `anon` comme pour le mode cloud, puis relance l'application.

Ne mets jamais la clé OpenAI dans `.env.local`, dans une variable `VITE_*` ou dans le code client. L'entrée vocale dépend de la prise en charge du navigateur ; la lecture des réponses utilise la synthèse vocale intégrée au navigateur.

## Déployer (lien public pour le jury)

```bash
npm run build
```

Puis sur vercel.com : **Add New → Project**, importe le dépôt, et ajoute les deux variables `VITE_SUPABASE_*` dans **Settings → Environment Variables**.

## Ce qui est fait

| Fonction (cahier des charges) | Fichier |
| --- | --- |
| F1 Création / rejoindre / démo | `src/pages/Bienvenue.jsx`, `src/lib/db.js` |
| F2 Photo + OCR + vérification | `src/pages/Releve.jsx`, `src/lib/ocr.js` |
| F3 Jauge Pont Kizuna | `src/components/PontGauge.jsx`, `src/pages/Accueil.jsx` |
| F4 Projection, alerte, surcoût | `src/lib/calcul.js` (testé) |
| F5 Simulateur d'appareils | `src/pages/Simulateur.jsx` |
| F6 Kizuna (partage, temps réel, messages) | `src/pages/Kizuna.jsx`, `src/lib/db.js` |
| F7 Kakeibo | `src/pages/Kakeibo.jsx` |
| F8 PDF de réclamation, preuves (EXIF, SHA-256, date serveur) | `src/pages/Reclamation.jsx`, `src/lib/pdf.js`, `src/lib/preuve.js` |
| F9 Météo du réseau (simulée, 17 h–23 h) | `src/pages/Accueil.jsx` |
| PWA installable | `vite.config.js` |
| **Aaddad parle** (lecture vocale FR / arabe) | `src/lib/voix.js`, bouton haut-parleur de l'Accueil |
| **Courbe du mois** (relevés + projection + seuil) | `src/components/Courbe.jsx` |
| **Alerte canicule** (≥ 35 °C dans les 3 jours, Open-Meteo) | `src/lib/meteo.js` |
| **Partage WhatsApp** (alerte famille, invitation Kizuna) | `src/pages/Accueil.jsx`, `src/pages/Kizuna.jsx` |
| **Arabe / derja** sur l'écran d'accueil | `src/lib/i18n.js` (ajouter une clé dans `fr` et `ar` pour traduire un nouveau texte) |
| **Grands textes** (réglage dans Kizuna) | `src/pages/Kizuna.jsx`, `html.grand` dans `app.css` |
| **Rappel de relevé** (plus de 7 jours) | `src/pages/Accueil.jsx` |
| **Version web** (barre latérale sur ordinateur) | `src/styles/app.css`, section « VERSION WEB » |

## À faire par l'équipe

- [ ] **Vérifier la grille tarifaire** dans `src/lib/tarifs.json` sur une vraie facture STEG. Le modèle actuel reproduit l'exemple publié par Webdo en 2022 (200 kWh ≈ 49,2 D ; 201 kWh ≈ 77,6 D), avec un prix moyen par régime.
- [ ] Vérifier les puissances des appareils du simulateur (même fichier).
- [ ] Tester l'OCR sur des photos de vrais compteurs de l'équipe ; ajuster le cadre dans `src/lib/ocr.js` et `.camera .frame` (`src/styles/app.css`) si besoin.
- [ ] Traduire les autres écrans en arabe (l'Accueil est fait : suivre le même modèle avec `useT()` et `dir={dir}`).
- [ ] Notifications push (Could) et QR code Kizuna (Could).

## Répartition (cahier des charges)

- **M1 — Front** : `src/pages/*`, `src/components/*`, `src/styles/*`.
- **M2 — Back, calculs, Kizuna** : `src/lib/calcul.js`, `src/lib/db.js`, `src/lib/store.jsx`, `supabase/schema.sql`.
- **M3 — OCR, preuves, PDF, pitch** : `src/lib/ocr.js`, `src/lib/preuve.js`, `src/lib/pdf.js`.

## Charte

Couleurs et typographies : `src/styles/tokens.css`, généré depuis les jetons de la charte Mieru. Images : `public/assets/` (logo, Aaddad en trois humeurs, illustration du pont Kizuna, motifs seigaiha et zellige).

---

## Version finale : langues, graphique, canicule, espace STEG

| Fonction | Où | Notes |
|---|---|---|
| **Choix de la langue FR / EN / AR** | 1er écran, Bienvenue, Kizuna, barre latérale | `src/i18n/` (fr.js, en.js, ar.js — mêmes clés). L'arabe passe toute l'app en droite-à-gauche. |
| **Graphique du mois** | Accueil | `components/Graphique.jsx` : index relevés, projection, seuil 200 kWh. |
| **Alerte canicule (3 jours)** | Accueil | `lib/meteo.js` (Open-Meteo, sans clé) + `components/Canicule.jsx`. Ville choisie dans Kizuna. |
| **Partage avec la STEG** | Kizuna → « Partager avec la STEG » | `components/PartageSteg.jsx`. Partage anonyme par défaut ou avec le nom du foyer ; partage révocable. |
| **Tableau de bord STEG** | `#/steg` (lien « Espace STEG (démo) » sur l'écran d'accueil) | `pages/Steg.jsx`, `components/StegCharts.jsx`, logique testée dans `lib/steg.js`. |
| **Photovoltaïque** | Carte sur l'accueil | `components/Photovoltaique.jsx` estime la production avec PVGIS. La puissance et les relevés restent sur l'appareil ; la contribution est facultative et le tableau n'expose que les agrégats par ville à partir de cinq foyers. |

### Règles de confidentialité du tableau de bord STEG (appliquées dans `lib/steg.js`, testées)
- Les données photovoltaïques ne sont transmises qu’après activation explicite de la contribution agrégée et du partage STEG.
- Jamais transmis au tableau : prénoms, code Kizuna, photos, messages, kakeibo, adresse. Le nom du compteur n’est envoyé que pour le partage nominatif.
- Zones de moins de 5 foyers masquées (k-anonymat).
- Les contributions photovoltaïques individuelles sont protégées par RLS ; l'export agrégé est lui aussi masqué tant que moins de cinq mesures existent.
- La production PV est une estimation mensuelle PVGIS proratisée, pas une mesure d'onduleur. L'injection réseau n'est affichée que si l'utilisateur la saisit comme mesurée.
- La liste « foyers en forte hausse » (+20 %) ne contient que les foyers « accepter un conseil », sous un code anonyme `MIE-XXXX`.
- Données simulées (480 foyers) clairement signalées, désactivables par une case.
- Avant un vrai déploiement : déclaration préalable à l'INPDP (loi organique n° 2004-63), accès au tableau de bord réservé à un compte STEG authentifié (voir fin de `supabase/schema.sql`).

En mode Supabase, exécuter à nouveau `supabase/schema.sql` pour ajouter les tables et vues de partage et photovoltaïque. Le tableau `/steg` reste une démo non authentifiée : les agrégats sont anonymisés, mais l'accès n'est pas réservé techniquement au personnel STEG. La soumission anonyme ne garantit pas l'authenticité des mesures ; un déploiement réel doit ajouter l'authentification et l'autorisation des foyers.

Tests : `npm test` (calculs, météo, STEG et photovoltaïque).

> Fichiers anciens non utilisés : `src/lib/i18n.js` et `src/components/Courbe.jsx` (remplacés par `src/i18n/` et `Graphique.jsx`). On peut les supprimer.

---

## Version « STEG connectée » : partage obligatoire, carte du réseau, délestage ciblé, solaire

| Fonction | Où | Fichiers |
|---|---|---|
| **Partage obligatoire avec la STEG** | Écran « Avant de commencer » (une fois par compteur) ; Kizuna → « Ce que la STEG voit » | `components/ConditionsSteg.jsx`, `components/PartageSteg.jsx` |
| **Besoin vital (équipement médical)** | Même écran : le logement n'est jamais limité ni coupé | `partages_steg.prioritaire` |
| **Carte zoomable** | `#/steg` → onglet « Carte et délestage » : Tunisie → gouvernorats → secteurs → maisons (molette, pincement, + / −) | `components/CarteSteg.jsx` (SVG, hors connexion) |
| **Délestage ciblé** | Clic sur un secteur → « Préparer le plan » : limiter à 2 kVA d'abord, couper seulement si besoin, prioritaires protégés, comparaison avec la coupure de tout le secteur | `lib/reseau.js` → `planDelestage` |
| **Action sur une maison** | Clic sur une maison : alerter, limiter, couper (confirmation), rétablir | `pages/Steg.jsx` |
| **Alertes et annonces côté foyer** | Bandeau STEG en haut de l'accueil (alerte, limitation/coupure annoncée 15 min avant, en cours, rétablie), lisible à voix haute | `components/AvisSteg.jsx`, table `avis_steg` |
| **Solaire clair** | Accueil → « Mon solaire » : produit / consommé, d'où vient l'électricité, où va la production, économies du foyer et de la STEG | `components/Photovoltaique.jsx` |
| **Gain STEG par maison solaire** | `#/steg` → onglet « Solaire » | `lib/reseau.js` → `gainsSolaire`, `syntheseSolaire` |

**Calcul du gain solaire** (facturation nette) : gain foyer = facture sans panneaux − facture avec panneaux ; gain STEG = production × 0,472 D/kWh (coût moyen, audition parlementaire 2025) − gain foyer.

**Données de démo** : 1 226 maisons simulées dans 31 secteurs (noms de quartiers réels, positions approximatives), plus les vrais foyers Mieru (marqués « Foyer Mieru »). Bouton « Pointe canicule 15 h » pour simuler l'été.

**Supabase** : exécuter à nouveau `supabase/schema.sql` (colonnes `prioritaire`, `pv_kwc`, `pv_injection_kwh`, vue `steg_foyers`, table `avis_steg`). La partie cloud de cette version n'a pas été testée contre un vrai projet Supabase.

**Avant un vrai déploiement** : convention foyer–STEG (loi organique 2004-63, art. 29) et déclaration à l'INPDP ; tableau de bord réservé à un compte STEG authentifié ; une vraie limitation ou coupure à distance demande des compteurs communicants.

Tests : `npm test` (37 tests, dont 11 pour le réseau et le délestage).
