<div align="center">

<img src="public/assets/mieru-mark.svg" alt="Logo Mieru" width="96">

# Mieru 見える — le pont Kizuna

**Voir chaque maison, pour ne plus plonger tout un quartier dans le noir.**

Application web progressive (PWA) réalisée en 24 h pendant le **hackathon Kizuna 🇹🇳 × 🇯🇵**.
Elle lit le compteur électrique des familles tunisiennes, les prévient avant le seuil des 200 kWh, et donne à la STEG une carte maison par maison pour cibler au lieu de couper.

`React` · `Vite` · `PWA` · `Supabase` · `Tesseract.js` · `Open-Meteo` · `PVGIS` · FR / EN / AR

### 👉 [Essayer l'application en ligne](https://hackaton-kizuna.vercel.app) · [Espace STEG (démo)](https://hackaton-kizuna.vercel.app/#/steg)

</div>

---

## Sommaire

1. [Le problème](#le-problème)
2. [Notre solution](#notre-solution)
3. [Fonctionnalités](#fonctionnalités)
4. [La touche japonaise](#la-touche-japonaise)
5. [Démarrage rapide](#démarrage-rapide)
6. [Modes de données et Supabase](#modes-de-données-et-supabase)
7. [Assistant IA Aaddad](#assistant-ia-aaddad)
8. [Déploiement](#déploiement)
9. [Tests](#tests)
10. [Structure du projet](#structure-du-projet)
11. [Confidentialité et cadre légal](#confidentialité-et-cadre-légal)
12. [Limites connues et suite](#limites-connues-et-suite)
13. [Équipe](#équipe)

---

## Le problème

Été 2026 : pendant la canicule, la STEG a multiplié les coupures tournantes, secteur par secteur, dans tout le Grand Tunis.

| Chiffre | Source |
|---|---|
| Pointe annoncée à **plus de 6 000 MW** le 21 juillet 2026 (record précédent : 4 888 MW en 2024) | Webdo |
| Capacité de production d'environ **4 630 MW**, soit **≈ 400 MW manquants** | allAfrica |
| **2,7 millions** de climatiseurs ; la clim représente 18 % de la consommation des ménages | ANME / STEG |
| Coût de production **456 millimes/kWh**, prix moyen de vente **291 millimes** | Tuniscope |
| **85 %** des abonnés consomment moins de 200 kWh par mois | Webdo |

**Le constat :** la STEG voit des secteurs, pas des maisons. Alors elle coupe des secteurs entiers, y compris les foyers qui économisent et les personnes malades.

Côté foyer, un deuxième piège : au-delà de **200 kWh par mois, toute la consommation passe au tarif haut** (environ +28 D sur la facture).

## Notre solution

Trois verbes :

- **見る Voir.** Le foyer photographie son compteur. Mieru lui dit combien il lui reste avant 200 kWh, et ce que la chaleur va lui coûter.
- **繋ぐ Relier.** Les relevés remontent à la STEG, qui obtient une carte de chaque secteur et de chaque maison.
- **守る Protéger.** En cas de surcharge, la STEG limite d'abord les plus gros consommateurs, prévient 15 minutes avant et ne touche jamais aux foyers ayant un équipement médical.

> Dans notre simulation du secteur du Bardo à 105 % de sa capacité : **3 maisons limitées à 2 kVA au lieu de 65 maisons coupées.**

## Fonctionnalités

### Pour le foyer

| Fonction | Description | Fichiers |
|---|---|---|
| Création, invitation, démo | Créer un compteur, rejoindre celui de la famille, ou tester la démo « Maison Ben Salah » | `src/pages/Bienvenue.jsx`, `src/lib/db.js` |
| Relevé par photo | Photo du compteur, lecture de l'index par OCR, vérification | `src/pages/Releve.jsx`, `src/lib/ocr.js` |
| Jauge « Pont Kizuna » | Aaddad avance de la porte de Sidi Bou Saïd vers le torii des 200 kWh | `src/components/PontGauge.jsx` |
| Projection et surcoût | Jour estimé du dépassement, surcoût en dinars | `src/lib/calcul.js` |
| Graphique du mois | Relevés, projection, seuil | `src/components/Graphique.jsx` |
| Alerte canicule | Météo des 3 prochains jours (35 °C et plus) et kWh supplémentaires de la clim | `src/lib/meteo.js`, `src/components/Canicule.jsx` |
| Avis de la STEG | Alerte de surconsommation, limitation ou coupure annoncée, retour du courant | `src/components/AvisSteg.jsx` |
| Mon solaire | Production, consommation réelle, économies du foyer et de la STEG (PVGIS, ou rendement moyen hors connexion) | `src/components/Photovoltaique.jsx`, `src/lib/pv.js` |
| Simulateur d'appareils | Ce que coûte chaque appareil | `src/pages/Simulateur.jsx` |
| Kakeibo | Le bilan japonais du mois : une seule amélioration choisie | `src/pages/Kakeibo.jsx` |
| Kizuna | Compteur partagé par la famille, en temps réel, avec messages et invitation WhatsApp | `src/pages/Kizuna.jsx` |
| Réclamation PDF | Photos datées et scellées (EXIF, SHA-256, date serveur) | `src/pages/Reclamation.jsx`, `src/lib/pdf.js`, `src/lib/preuve.js` |
| Accessibilité | Français, anglais, arabe (de droite à gauche), lecture vocale par Aaddad, grands textes, rappel de relevé après 7 jours | `src/i18n/`, `src/lib/voix.js` |

### Pour la STEG (`#/steg`)

| Fonction | Description | Fichiers |
|---|---|---|
| Carte zoomable | Tunisie → gouvernorats → secteurs → maisons, charge de chaque secteur | `src/components/CarteSteg.jsx` |
| Délestage ciblé | Plan qui limite avant de couper, protège les foyers prioritaires et se compare à la coupure de tout le secteur | `src/lib/reseau.js` (`planDelestage`) |
| Actions sur une maison | Alerter, limiter à 2 kVA, couper (avec confirmation), rétablir | `src/pages/Steg.jsx` |
| Solaire | Gain par maison équipée, pour le foyer et pour la STEG | `src/lib/reseau.js` (`gainsSolaire`, `syntheseSolaire`) |
| Statistiques | Par gouvernorat, par tranche de kWh, chaleur et consommation, foyers en forte hausse | `src/components/StegCharts.jsx`, `src/lib/steg.js` |
| Scénario | Bouton « Pointe canicule 15 h » pour simuler l'été | `src/pages/Steg.jsx` |

**Calcul du gain solaire (facturation nette)**
- Gain du foyer = facture sans panneaux − facture avec panneaux.
- Gain de la STEG = production × 0,472 D/kWh (coût moyen) − gain du foyer.

## La touche japonaise

| | Concept | Dans Mieru |
|---|---|---|
| 見える化 | *Mieruka*, « rendre visible » | Le nom et le principe de l'application |
| 節電 | *Setsuden*, l'économie d'électricité de 2011 au Japon | Le délestage ciblé et le « Défi Setsuden » entre quartiers |
| 家計簿 | *Kakeibo*, le carnet de comptes des familles | Le bilan du mois |
| 絆 | *Kizuna*, le lien | Le compteur partagé en famille, et le pont Tunisie–Japon |
| 判子 | *Hanko*, le sceau | Les photos scellées de la réclamation |

**Aaddad (عدّاد, « compteur »)** est notre mascotte : un compteur coiffé d'un torii (le seuil des 200 kWh), avec une porte de Sidi Bou Saïd pour ventre et des joues de sakura. Il a trois humeurs : calme, inquiet, joyeux.

## Démarrage rapide

**Prérequis :** Node.js 18 ou plus récent.

```bash
git clone https://github.com/iness33/hackaton_Kizuna.git
cd hackaton_Kizuna
npm install
npm run dev
```

Ouvrir l'adresse affichée (par exemple `http://localhost:5173`).

- **Sur téléphone :** même Wi-Fi que l'ordinateur, puis l'adresse « Network » affichée par Vite.
- **Caméra :** elle exige HTTPS ou `localhost`. Pour un HTTPS rapide, rediriger le port 5173 depuis l'onglet **PORTS** de VS Code en visibilité *Public*. Sur téléphone, le bouton de choix de photo ouvre de toute façon l'appareil photo.
- **Démo :** le bouton « Essayer avec la démo Maison Ben Salah » crée un compteur de 3 membres avec un mois de relevés (168 kWh) et 3 kWc de panneaux solaires.
- **Espace STEG :** lien « Espace STEG (démo) » sur l'écran d'accueil, ou `#/steg`.

## Modes de données et Supabase

| Mode | Condition | Fonctionnement |
|---|---|---|
| **Local** | Par défaut, sans configuration | Tout fonctionne sur un seul appareil (localStorage et IndexedDB). Kizuna reste limité à cet appareil. |
| **Cloud** | Fichier `.env.local` rempli | En plus : Kizuna en temps réel entre téléphones, photos stockées sur Supabase, données transmises à l'espace STEG. |

### Activer Supabase (environ 10 minutes)

1. Créer un projet gratuit sur [supabase.com](https://supabase.com).
2. Dans **SQL Editor**, exécuter `supabase/schema.sql` (tables, bucket de photos, temps réel, tables STEG `partages_steg` et `avis_steg`, vue `steg_foyers`).
3. Dans **Project Settings → API**, copier l'URL du projet et la clé `anon public`.
4. Copier `.env.example` en `.env.local` et remplir :
   ```
   VITE_SUPABASE_URL=...
   VITE_SUPABASE_ANON_KEY=...
   ```
5. Relancer `npm run dev`. Les comptes créés en mode local ne sont pas transférés.

> ⚠️ Pour le hackathon, les règles d'accès sont volontairement simplifiées : toute personne qui connaît l'identifiant d'un compteur peut le lire et y écrire.

## Assistant IA Aaddad

L'assistant passe par une **Supabase Edge Function** qui appelle OpenAI côté serveur. Les relevés du compteur ne sont pas envoyés à OpenAI et les conversations ne sont pas enregistrées. Limite : 20 messages par jour et par adresse réseau.

1. Exécuter `supabase/assistant.sql` dans le SQL Editor.
2. Installer la Supabase CLI, puis :
   ```bash
   supabase link --project-ref <project-ref>
   supabase secrets set "OPENAI_API_KEY=ta-cle"
   supabase functions deploy aaddad-assistant
   ```
3. Remplir `.env.local` (URL et clé `anon`), puis relancer l'application.

> 🔒 La clé OpenAI ne doit **jamais** se trouver dans `.env.local`, dans une variable `VITE_*` ni dans le code client.

## Déploiement

L'application est en ligne sur Vercel : **https://hackaton-kizuna.vercel.app** (mode local : les données restent sur l'appareil de chaque visiteur).

Chaque `git push` sur `main` redéploie automatiquement. Pour refaire un déploiement depuis zéro :

```bash
npm run build
```

Puis sur [vercel.com](https://vercel.com) : **Add New → Project**, importer le dépôt (Vite est détecté automatiquement). Pour le mode cloud, ajouter `VITE_SUPABASE_URL` et `VITE_SUPABASE_ANON_KEY` dans **Settings → Environment Variables**. Vercel fournit le HTTPS nécessaire à la caméra.

## Tests

```bash
npm test
```

37 tests (Node.js `node:test`) :

- **Calculs** (`calcul.test.js`) : coût mensuel, seuil, projection, vérification des relevés.
- **Météo** (`meteo.test.js`) : alerte canicule et kWh supplémentaires.
- **Photovoltaïque** (`pv.test.js`).
- **Tableau STEG** (`steg.test.js`) : agrégation et règles de confidentialité.
- **Réseau et délestage** (`reseau.test.js`) : on limite avant de couper, les foyers prioritaires ne sont jamais touchés, une action est annoncée avant et dure 60 minutes au plus, gains solaires.

## Structure du projet

```
hackaton_Kizuna/
├── public/assets/        # Logo, Aaddad (3 humeurs), pont Kizuna, motifs seigaiha et zellige
├── src/
│   ├── pages/            # Bienvenue, Accueil, Releve, Simulateur, Kakeibo, Kizuna, Reclamation, Steg
│   ├── components/       # PontGauge, Graphique, Canicule, Photovoltaique, AvisSteg,
│   │                     # ConditionsSteg, PartageSteg, CarteSteg, StegCharts, Chrome, Icon
│   ├── lib/              # calcul, reseau, steg, pv, meteo, ocr, preuve, pdf, voix, db, store
│   │   └── tarifs.json   # Grille tarifaire et puissance des appareils
│   ├── i18n/             # fr.js, en.js, ar.js
│   └── styles/           # tokens.css (charte), app.css
├── supabase/             # schema.sql, assistant.sql, fonction Edge aaddad-assistant
├── .env.example
├── index.html
├── package.json
└── vite.config.js        # PWA (manifest, service worker)
```

## Confidentialité et cadre légal

- **Partage avec la STEG :** condition d'utilisation, acceptée une fois par compteur sur l'écran « Avant de commencer ». Le foyer voit à tout moment ce que la STEG voit (Kizuna → « Ce que la STEG voit »).
- **Jamais transmis :** prénoms des membres, photos, messages Kizuna, kakeibo.
- **Foyers prioritaires :** un foyer qui déclare un équipement médical n'est jamais limité ni coupé.
- **Délestage :** toujours annoncé 15 minutes avant, 60 minutes au plus, limitation à 2 kVA avant toute coupure.
- **Base légale visée :** convention entre le foyer et la STEG (loi organique n° 2004-63, art. 29), avec déclaration préalable à l'INPDP avant tout déploiement réel.
- **Données de démo :** les 1 226 maisons de la carte, leurs positions et leurs charges sont **simulées**. Les vrais foyers Mieru sont marqués « Foyer Mieru ».

## Limites connues et suite

- Vérifier la grille tarifaire (`src/lib/tarifs.json`) sur une vraie facture STEG.
- Tester l'OCR sur de vraies photos de compteurs et ajuster le cadre (`src/lib/ocr.js`, `.camera .frame` dans `app.css`).
- La partie cloud de l'espace STEG n'a pas encore été testée sur un vrai projet Supabase.
- Une limitation ou une coupure **à distance** nécessite des compteurs communicants. Sans eux, Mieru alerte et la STEG agit sur le terrain.
- À venir : notifications push, QR code d'invitation Kizuna, pilote dans un secteur avec la STEG.
- `src/lib/i18n.js` et `src/components/Courbe.jsx` sont obsolètes et peuvent être supprimés.

## Équipe

Projet réalisé pendant le hackathon Kizuna (Tunisie × Japon).

| Rôle | Membre | Périmètre |
|---|---|---|
| M1 · Front | [Nom] | `src/pages/*`, `src/components/*`, `src/styles/*` |
| M2 · Back, calculs, Kizuna | [Nom] | `src/lib/calcul.js`, `src/lib/db.js`, `src/lib/store.jsx`, `supabase/` |
| M3 · OCR, preuves, PDF, pitch | [Nom] | `src/lib/ocr.js`, `src/lib/preuve.js`, `src/lib/pdf.js` |

---

<div align="center">

**Mieru 見える : voir, pour ne plus couper.**

ありがとう · شكرا · Merci

</div>
