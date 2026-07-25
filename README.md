# Now Playing — Overlay musique pour OBS (multi-lecteurs)

Carte « now playing » élégante pour OBS : pochette (ou vidéo Canvas Spotify), titre,
artiste, barre d'avancement fluide et temps. Se masque toute seule en pause. Pensée pour
être **fiable d'abord** : si une brique optionnelle échoue (canvas), l'overlay retombe
proprement sur la pochette — jamais de panne visible à l'écran.

Fichier unique : `spotify-nowplaying.html` — aucune installation, aucune dépendance en ligne
pour le mode démo.

## Démarrage rapide
1. **Voir la démo** (sans rien installer) : ouvrez le fichier dans un navigateur en ajoutant
   `?test=1` à la fin de l'adresse.
2. **Régler l'apparence sans lire la doc** : ouvrez **`configurateur.html`** — vous choisissez
   les couleurs, la taille et le comportement, l'aperçu se met à jour en direct, et vous n'avez
   plus qu'à copier l'adresse générée pour OBS.
3. **En vrai** : la carte s'alimente via le **serveur WebSocket de Streamer.bot**
   (127.0.0.1:8080) — voir « Branchement » plus bas.
4. **Dans OBS** : source Navigateur → « Fichier local » → ce fichier. ~300×520, fond
   transparent (géré).

## Personnalisation — tout se règle dans l'URL, sans toucher au code
| Paramètre | Effet | Exemple |
|---|---|---|
| `?layout=horizontal` | Bandeau : pochette à gauche, textes à droite | |
| `?layout=mini` | Bandeau minimaliste (sans temps chiffrés) | |
| `?layout=text` | Texte seul, sans pochette | |
| `?cover=84` | Taille de la pochette en bandeau (px) | `?layout=mini&cover=52` |
| `?w=280` | Largeur de la carte (px, défaut 240 ; 340 en bandeau) | `?w=200` |
| `?media=square` \| `?media=portrait` | Format de la zone média (carte verticale uniquement, défaut 3:4) | `?media=square` |
| `?accent=RRGGBB` | Couleur d'accent (barre) | `?accent=00B4FF` |
| `?text=RRGGBB` | Couleur des textes | `?text=ffffff` |
| `?panel=RRGGBB` + `?panelalpha=0.9` | Fond de la carte | `?panel=101018` |
| `?radius=18` | Arrondi des coins (px) | `?radius=8` |
| `?font=Inter` | Police Google Fonts | `?font=Montserrat` |
| `?hidePaused=3000` | Masquer X ms après une pause (0 = jamais) | `?hidePaused=0` |
| `?offset=1500` | Décalage de la barre en ms (positif = rattrape un retard) | `?offset=-500` |
| `?align=center` \| `?align=right` | Alignement des textes | |
| `?bar=0` / `?times=0` | Masquer la barre / les temps chiffrés | `?times=0` |
| `?shadow=0` | Sans ombre portée | |
| `?blur=22` | Intensité du flou du fond (carte verticale) | `?blur=0` |
| `?clientId=` | **Mode direct** : identifiant de votre app Spotify | |
| `?poll=2000` | Mode direct : fréquence d'interrogation (ms, min. 1000) | `?poll=1500` |
| `?host=` / `?port=` | Adresse du WebSocket Streamer.bot | `?port=8081` |
| `?debug=1` | Panneau de diagnostic (connexion + messages) | |
| `?test=1` | Démo autonome (fausses pistes locales) | |

Les paramètres se combinent : `...html?w=260&accent=ff4f9a&font=Inter&hidePaused=0`

## Deux façons de faire fonctionner l'overlay

**A. Mode direct (recommandé)** — le widget parle à Spotify tout seul : aucun logiciel
intermédiaire à installer (ni Streamer.bot, ni Mix It Up), et la barre de progression se
resynchronise en permanence (même si vous déplacez la lecture à la main).
**Contrainte** : la page doit être servie en HTTPS — voir ci-dessous.

**B. Mode Streamer.bot** — le widget reçoit les informations d'une extension via le serveur
WebSocket local. Fonctionne depuis un simple fichier sur le disque, mais demande plus
d'installation, et la position n'est rafraîchie qu'aux changements de morceau / pause /
reprise.

## Mode direct — installation
1. **Hébergez le fichier** sur une page statique HTTPS (GitHub Pages, Netlify, Cloudflare
   Pages… tous gratuits). Vous obtenez une adresse du type
   `https://votre-nom.github.io/overlay/spotify-nowplaying.html`.
   ⚠ Un fichier ouvert directement depuis le disque (`file://`) **ne peut pas** fonctionner
   en mode direct : Spotify n'accepte que des adresses de retour en https.
2. **Créez votre app** sur [developer.spotify.com](https://developer.spotify.com/dashboard)
   → *Create app*. Dans **Redirect URI**, collez **exactement l'adresse de l'étape 1**
   (sans paramètres). Cochez « Web API ». Notez le **Client ID** (le *secret* est inutile ici).
3. **Ouvrez l'overlay** avec votre identifiant :
   `https://…/spotify-nowplaying.html?clientId=VOTRE_CLIENT_ID`
   La première fois, Spotify demande votre accord ; ensuite, la connexion est mémorisée et
   renouvelée automatiquement.
4. **Dans OBS** : ajoutez cette même adresse en source **Navigateur**. Faites l'autorisation
   **depuis OBS** (clic droit sur la source → *Interagir*) : la mémoire du navigateur d'OBS
   est séparée de celle de votre navigateur habituel.

À savoir : Spotify Premium est requis, le widget ne demande que des **permissions de
lecture** (il ne peut pas commander votre musique), et l'interrogation se fait toutes les
2 s par défaut (`?poll=` pour ajuster, minimum 1000 ms).

## Branchement (Streamer.bot + extension Spotify de Tawmae)
1. Installez « Spotify & SB » (tawmae.xyz/spotify-and-sb) — nécessite Spotify Premium et une
   app sur developer.spotify.com (Redirect URI exact : `http://127.0.0.1:1312/tawmaeSpotify/`).
2. Streamer.bot → Servers/Clients → WebSocket Server : Auto Start, 127.0.0.1:8080.
3. Créez une action (ex. « Spotify Overlay ») avec les 3 triggers de l'extension
   (New Song, Song Paused, Song Continued) et **une seule sous-action** :
   **Core → Execute C# Code**, contenant exactement :

```csharp
using System;

public class CPHInline
{
    public bool Execute()
    {
        string g(string key){ return CPH.TryGetArg(key, out string v) ? (v ?? "") : ""; }
        string esc(string s){ return (s ?? "").Replace("\\", "\\\\").Replace("\"", "\\\""); }

        string json =
            "{\"widget\":\"kickaxe-music\","
          + "\"trackId\":\""       + esc(g("trackId"))       + "\","
          + "\"trackName\":\""     + esc(g("trackName"))     + "\","
          + "\"artists\":\""       + esc(g("artists"))       + "\","
          + "\"coverImageURL\":\"" + esc(g("coverImageURL")) + "\","
          + "\"isPlaying\":\""     + esc(g("isPlaying"))     + "\","
          + "\"progressMs\":\""    + esc(g("progressMs"))    + "\","
          + "\"durationMs\":\""    + esc(g("durationMs"))    + "\"}";

        CPH.WebsocketBroadcastJson(json);
        return true;
    }
}
```

⚠ La sous-action « Custom Event Trigger » de Streamer.bot ne diffuse **pas** sur le WebSocket
(elle ne déclenche que du code interne) : c'est bien `CPH.WebsocketBroadcastJson` qu'il faut.

## Dépannage
Ouvrez l'overlay dans un navigateur avec **`?debug=1`** : un encart affiche l'état de la
connexion, le nombre de messages reçus/reconnus et le dernier message brut.

| Symptôme | Cause probable | Solution |
|---|---|---|
| `DÉCONNECTÉ` | WebSocket Server de Streamer.bot arrêté ou autre port | Servers/Clients → WebSocket Server → Start + Auto Start ; sinon `?port=` |
| `CONNECTÉ`, reçus = 0 | L'action ne se déclenche pas, ou sous-action incorrecte | Testez l'action à la main (Test Trigger) ; vérifiez que la sous-action est bien **Execute C# Code** |
| Reçus > 0, reconnus = 0 | Format inattendu | Le message brut est affiché sous les tirets — comparez-le au contrat |
| Champs vides après un Test Trigger | Normal | Un test manuel n'a pas de morceau : changez de piste dans Spotify |
| Rien à l'écran, tout est vert | La carte est masquée (pause/silence) | Lancez la lecture, ou `?hidePaused=0` |

## Autres lecteurs (Apple Music, YouTube Music, Deezer…)
Le widget est **source-agnostique** : n'importe quelle source capable d'envoyer le contrat
JSON ci-dessous sur le WebSocket de Streamer.bot fonctionne. Il n'existe pas de méthode
unique côté « capture » : selon le lecteur, passez par le plugin Tuna (OBS), la session
média Windows, ou une intégration dédiée — la barre de progression n'est pas toujours
disponible selon la source.

```json
{ "widget": "kickaxe-music", "trackId": "…", "trackName": "…", "artists": "…",
  "coverImageURL": "https://…", "isPlaying": "True", "progressMs": "1591",
  "durationMs": "236960", "canvasUrl": "" }
```
(Toutes les valeurs peuvent être des chaînes ; `kind` optionnel : nowplaying/paused/resumed.)

## Canvas Spotify (vidéos) — support présent, mais INEXPLOITABLE à ce jour
⚠️ **Testé le 25/07/2026 : ça ne fonctionne pas, et ce n'est pas réparable durablement.**
Spotify n'offre aucune API officielle pour les canvas ; tous les projets communautaires
passent par un jeton généré à partir d'un secret interne que **Spotify fait tourner en
permanence** (plus de 50 versions ; la version peut changer plusieurs fois par jour).
Un service à jour aujourd'hui peut casser demain, en plein direct. C'est aussi ce qui fait
tomber les widgets canvas commerciaux.

Le widget garde le support technique (au cas où la situation changerait) :
- `?canvasApi=<url>` : le widget interroge un micro-service (trackId ajouté en fin d'URL, ou
  à la place de `%id%`) et lit l'URL `.mp4` dans la réponse (plusieurs formats acceptés).
- Ou `canvasUrl` fourni directement dans le contrat.
- `?canvasTest=<un.mp4>` : force une vidéo sur tous les morceaux — utile pour juger du rendu.

Dans tous les cas, **échec = repli pochette silencieux** : l'overlay ne tombe jamais.

## Crédits / contexte
**Un projet de Kickaxe** · [twitch.tv/kick4xe](https://twitch.tv/kick4xe) · **StreamUp** · 2026
Conception et direction de projet : Jérémy (Kickaxe).

Partagé à titre personnel : proches qui le demandent, et participants du cursus **StreamUp**.
Ce n'est pas un produit public. Merci de conserver la mention d'origine et de citer la source
en cas de partage — c'est la seule contrepartie demandée.

Pensé après les pannes répétées d'un widget tiers : la fiabilité prime, tout le reste est
optionnel. Détails techniques et décisions : `_NOTES.md`.
