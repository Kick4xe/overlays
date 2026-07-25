# StreamUp : Now Playing - Twitch Widget

Petite carte qui affiche la musique en cours pendant le stream : pochette, titre, artiste,
barre de progression et temps. Elle disparaît toute seule quand la lecture est en pause.

J'ai fait ce widget parce que celui que j'utilisais avant tombait régulièrement en panne en
plein live. Ici, le principe c'est : ça marche, ou au pire ça affiche moins de choses, mais
ça ne casse pas.

Un seul fichier : `spotify-nowplaying.html`. Rien à installer.

## Par où commencer

Tout se passe ici : **https://github.com/Kick4xe/streamup-now_playing-twitch_widget**

**Le configurateur en ligne** (le plus simple, rien à télécharger pour regarder) :
https://kick4xe.github.io/streamup-now_playing-twitch_widget/configurateur.html

**L'overlay en ligne** (celui que vous mettrez dans OBS) :
https://kick4xe.github.io/streamup-now_playing-twitch_widget/spotify-nowplaying.html

Deux façons de l'utiliser :

- **Utiliser ma version hébergée.** Vous ne téléchargez rien. Vous créez votre app Spotify
  et vous mettez **mon adresse** comme Redirect URI (celle de l'overlay, juste au-dessus).
  Ça marche : chacun autorise son propre compte, rien ne passe par moi et je ne vois rien de
  vos données. C'est le chemin le plus court.
- **Héberger votre propre copie.** Bouton vert **Code** > **Download ZIP** sur le dépôt, puis
  vous remettez les fichiers sur votre propre GitHub Pages. À préférer si vous voulez rester
  indépendant de mon dépôt, ou modifier le code.

Dans les deux cas, la suite est la même : suivez la marche à suivre plus bas.

## Voir à quoi ça ressemble

Ouvrez le fichier dans un navigateur en ajoutant `?test=1` à la fin de l'adresse : de fausses
pistes défilent, sans avoir besoin de Spotify.

Pour régler l'apparence, ouvrez `configurateur.html` : vous cliquez, l'aperçu se met à jour,
et l'adresse à coller dans OBS se construit toute seule.

## Le faire fonctionner avec Spotify

Deux méthodes. La première est plus simple, prenez-la si vous hésitez.

### Méthode 1 : sans logiciel supplémentaire (recommandée)

Le widget parle à Spotify tout seul. Pas de Streamer.bot, pas de Mix It Up, rien à installer.
Autre avantage : la barre de progression reste juste même si vous avancez dans un morceau à
la main.

Il faut par contre héberger le fichier sur une page en HTTPS, parce que Spotify refuse les
adresses de retour en local. GitHub Pages fait ça gratuitement, et ça prend cinq minutes.

1. Mettez `spotify-nowplaying.html` sur GitHub Pages (ou Netlify, ou Cloudflare Pages).
   Vous obtenez une adresse du genre
   `https://votrenom.github.io/streamup-now_playing-twitch_widget/spotify-nowplaying.html`.
2. Sur developer.spotify.com, créez une app. Dans « Redirect URI », collez exactement
   l'adresse de l'étape 1, sans rien après. Cochez « Web API ». Notez le Client ID
   (le Client secret ne sert pas ici).
3. Ouvrez votre adresse en ajoutant `?clientId=VOTRE_CLIENT_ID`. Spotify demande votre accord
   une fois, et c'est mémorisé ensuite.
4. Dans OBS : source Navigateur, collez la même adresse.

Il faut Spotify Premium (Spotify l'exige depuis février 2026 pour toutes les apps). Le widget
demande uniquement le droit de *lire* ce qui joue, il ne peut pas commander votre musique.

**Astuce pour éviter de se reconnecter dans OBS.** Le navigateur d'OBS a sa propre mémoire,
donc il redemande la connexion. Pour éviter ça : ouvrez l'adresse dans votre navigateur
habituel avec `&showToken=1` en plus, un encart affiche une adresse prête à coller dans OBS.
Elle contient votre connexion, donc gardez-la pour vous et ne l'affichez pas à l'écran.

### Méthode 2 : via Streamer.bot

Si vous utilisez déjà Streamer.bot, le widget peut recevoir les infos par son serveur
WebSocket. Ça fonctionne depuis un simple fichier sur le disque, mais il y a plus de choses à
configurer, et la position n'est mise à jour qu'aux changements de morceau, pause et reprise.

1. Installez l'extension « Spotify & SB » de Tawmae (tawmae.xyz/spotify-and-sb).
2. Streamer.bot, onglet Servers/Clients, WebSocket Server : Auto Start, 127.0.0.1:8080.
3. Créez une action avec les trois déclencheurs de l'extension (New Song, Song Paused,
   Song Continued) et une seule sous-action : Core > Execute C# Code, avec ceci dedans :

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

Attention : la sous-action « Custom Event Trigger » ne marche pas pour ça, elle n'envoie rien
sur le WebSocket. J'ai perdu une bonne heure dessus.

## Réglages

Tout se passe dans l'adresse. Le configurateur les génère pour vous, mais voici la liste.

| Paramètre | Ce que ça fait |
|---|---|
| `?layout=horizontal` | Bandeau : pochette à gauche, textes à droite |
| `?layout=mini` | Bandeau plus discret, sans les temps |
| `?layout=text` | Texte seul, sans pochette |
| `?layout=bare` | Une seule ligne, sans cadre ni fond |
| `?w=280` | Largeur en pixels |
| `?cover=84` | Taille de la pochette dans les bandeaux |
| `?media=square` ou `?media=portrait` | Format de la zone image (carte verticale) |
| `?accent=RRGGBB` | Couleur de la barre |
| `?text=RRGGBB` | Couleur des textes |
| `?panel=RRGGBB` + `?panelalpha=0.9` | Fond de la carte |
| `?radius=18` | Arrondi des coins |
| `?font=Inter` | Police (Google Fonts) |
| `?align=center` ou `?align=right` | Alignement des textes |
| `?bar=0` / `?times=0` | Masquer la barre / les temps |
| `?shadow=0` | Sans ombre portée |
| `?blur=22` | Flou du fond derrière la pochette |
| `?hidePaused=3000` | Masquer après X ms de pause (0 = jamais) |
| `?offset=1500` | Décale la barre si elle est en retard (méthode 2 seulement) |
| `?clientId=` | Votre Client ID Spotify (méthode 1) |
| `?poll=2000` | Fréquence de rafraîchissement en ms (1000 minimum) |
| `?debug=1` | Affiche un panneau de diagnostic |
| `?test=1` | Fait tourner de fausses pistes |

On peut les combiner : `...html?layout=mini&accent=00B4FF&clientId=xxxxx`

## Quand ça ne marche pas

Ouvrez l'overlay avec `?debug=1`. Le panneau indique la version, le mode utilisé et ce qui
est reçu.

- Le panneau affiche « DÉCONNECTÉ » : le serveur WebSocket de Streamer.bot n'est pas lancé,
  ou pas sur le bon port.
- Connecté mais rien ne se passe : testez l'action à la main dans Streamer.bot pour voir si
  elle se déclenche. Vérifiez aussi que c'est bien la sous-action C# et pas une autre.
- Vous venez de mettre le fichier à jour et rien ne change : c'est le cache. Testez en
  navigation privée en ajoutant `&v=2` à l'adresse. Dans OBS, faites « Actualiser le cache de
  la page en cours » sur la source.
- Rien à l'écran alors que tout semble vert : la carte se cache quand la musique est en pause.
  Lancez la lecture, ou mettez `?hidePaused=0`.

## Autres lecteurs que Spotify

Le widget se moque de la source : il affiche tout message qui respecte ce format, envoyé sur
le WebSocket de Streamer.bot.

```json
{ "widget": "kickaxe-music", "trackId": "...", "trackName": "...", "artists": "...",
  "coverImageURL": "https://...", "isPlaying": "True", "progressMs": "1591",
  "durationMs": "236960" }
```

Donc Deezer, Apple Music ou YouTube Music sont possibles en théorie. En pratique il faut
trouver quoi lit le morceau en cours sur votre machine (le plugin Tuna pour OBS, ou la session
média de Windows) et lui faire envoyer ce JSON. Je ne l'ai pas fait, donc je ne peux pas dire
que ça fonctionne. À noter aussi : selon la méthode, la position dans le morceau n'est pas
toujours disponible, et la barre devient alors inutile.

### Comment s'y prendre, concrètement

Si vous voulez tenter, voilà l'ordre dans lequel je m'y prendrais. La partie compliquée n'est
pas le widget, c'est de récupérer le morceau en cours.

**1. Trouver ce qui lit votre lecteur.** Deux pistes selon le cas :
- Le plugin **Tuna** pour OBS sait lire plusieurs sources et écrit le titre dans un fichier.
- La **session média de Windows** (celle qui fait marcher les touches lecture/pause du clavier)
  expose le morceau en cours quel que soit le logiciel. Il faut un petit script pour la lire.

**2. Vérifier ce qu'on récupère vraiment.** Avant d'aller plus loin, regardez si vous obtenez
le titre, l'artiste, la pochette, et surtout la **position dans le morceau**. Les deux premiers
sont presque toujours là, la pochette souvent, la position rarement. Sans position, mettez
`?bar=0&times=0` et vous aurez un affichage propre sans barre.

**3. Envoyer le JSON sur le WebSocket de Streamer.bot.** Reprenez l'action décrite dans la
méthode 2 plus haut, en remplaçant les variables de Tawmae par les vôtres. Le code C# est le
même, seuls les noms des arguments changent.

**4. Vérifier avec `?debug=1`.** Le panneau vous dira si le message arrive et s'il est compris.
S'il arrive mais n'est pas reconnu, le message brut s'affiche dessous : comparez-le au format
ci-dessus, c'est en général une histoire de nom de champ.

À savoir : le mode sans logiciel (méthode 1) est spécifique à Spotify, parce qu'il utilise
leur API. Pour les autres lecteurs, il faut forcément passer par Streamer.bot ou équivalent.

## Les vidéos Canvas de Spotify

J'ai essayé, ça ne marche pas et ce n'est pas réparable. Spotify n'a pas d'API officielle pour
les Canvas, et les solutions de contournement passent par un secret interne que Spotify change
en permanence, parfois plusieurs fois par jour. Un projet à jour aujourd'hui peut casser
demain en plein live. C'est d'ailleurs ce qui fait tomber les widgets commerciaux qui les
proposent.

Le code pour les afficher est resté dans le fichier au cas où ça changerait un jour
(`?canvasApi=`), mais je ne conseille pas de s'y lancer. La pochette suffit et ne tombe jamais.

## Crédits

Un projet **StreamUp**, par **Kickaxe** (Jérémy) · [twitch.tv/kick4xe](https://twitch.tv/kick4xe) · 2026

StreamUp, c'est mon activité d'accompagnement technique pour streamers : former plutôt que
faire à la place, pour que chacun devienne autonome sur sa propre technique. Ce widget est né
comme ça, en cherchant une solution à un problème que j'avais moi-même en live.

Partagé à titre personnel, pour des proches et pour les personnes accompagnées via StreamUp.
Ce n'est pas un produit public. Si vous le repartagez, gardez la mention d'origine, c'est tout
ce que je demande.
