---
summary: Apprenez à lire et à mettre à jour les valeurs de configuration dans AdonisJS.
---

# Configuration

Les fichiers de configuration de votre application AdonisJS sont stockés dans le répertoire `config`. Une nouvelle application AdonisJS est accompagnée de quelques fichiers préexistants utilisés par le noyau du framework et les packages installés.

N'hésitez pas à créer des fichiers supplémentaires dont votre application a besoin dans le répertoire `config`.


:::note

Nous vous recommandons d'utiliser les [variables d'environnement](./environment_variables.md) pour stocker les données sensibles et la configuration spécifique à l'environnement.


:::

## Importation des fichiers de configuration

Vous pouvez importer les fichiers de configuration dans le code de votre application en utilisant l'instruction `import` standard de JavaScript. Par exemple :

```ts
import { appKey } from '#config/app'
```

```ts
import databaseConfig from '#config/database'
```

## Utilisation du service de configuration

Le service de configuration propose une API alternative pour lire les valeurs de configuration. Dans l'exemple suivant, nous utilisons le service de configuration pour lire la valeur `appKey` stockée dans le fichier `config/app.ts`.

```ts
import config from '@adonisjs/core/services/config'

config.get('app.appKey')
config.get('app.http.cookie') // lire les valeurs imbriquées
```

La méthode `config.get` accepte une clé séparée par un point et l'analyse de la façon suivante.

- La première partie est le nom du fichier à partir duquel vous souhaitez lire les valeurs. Par exemple, le fichier `app.ts`.
- Le reste de la chaîne est la clé que vous souhaitez accéder parmi les valeurs exportées. Par exemple, `appKey` dans ce cas.

## Service de configuration vs. importation directe des fichiers de configuration

Utiliser le service de configuration plutôt que d'importer directement les fichiers de configuration ne présente pas d'avantages directs. Cependant, le service de configuration est le seul choix pour lire la configuration dans les packages externes et les templates Edge.

### Lecture de la configuration à l'intérieur des packages externes

Si vous créez un package tiers, vous ne devriez pas importer directement les fichiers de configuration depuis l'application utilisateur car cela rendra votre package étroitement lié à la structure de dossiers de l'application hôte.

Au lieu de cela, vous devriez utiliser le service de configuration pour accéder aux valeurs de configuration à l'intérieur d'un fournisseur de service. Par exemple :

```ts
import { ApplicationService } from '@adonisjs/core/types'

export default class DriveServiceProvider {
  constructor(protected app: ApplicationService) {}
  
  register() {
    this.app.container.singleton('drive', () => {
      // highlight-start
      const driveConfig = this.app.config.get('drive')
      return new DriveManager(driveConfig)
      // highlight-end
    })
  }
}
```

### Lecture de la configuration à l'intérieur des templates Edge

Vous pouvez accéder aux valeurs de configuration à l'intérieur des templates Edge en utilisant la méthode globale `config`.

```edge
<a href="{{ config('app.appUrl') }}"> Home </a>
```

Vous pouvez utiliser la méthode `config.has` pour vérifier si une valeur de configuration existe pour une clé donnée. La méthode renvoie `false` si la valeur est `undefined`.

```edge
@if(config.has('app.appUrl'))
  <a href="{{ config('app.appUrl') }}"> Home </a>
@else
  <a href="/"> Home </a>
@end
```

## Modification de l'emplacement de la configuration

Vous pouvez mettre à jour l'emplacement du répertoire de configuration en modifiant le fichier `adonisrc.ts`. Après la modification, les fichiers de configuration seront importés à partir du nouvel emplacement.

```ts
directories: {
  config: './configurations'
}
```

Assurez-vous de mettre à jour l'alias d'importation dans le fichier `package.json`.

```json
{
  "imports": {
    "#config/*": "./configurations/*.js"
  }
}
```

## Limitations des fichiers de configuration

Les fichiers de configuration stockés dans le répertoire `config` sont importés pendant la phase de démarrage de l'application. Par conséquent, les fichiers de configuration ne peuvent pas dépendre du code de l'application.

Par exemple, si vous essayez d'importer et d'utiliser le service de routage à l'intérieur du fichier `config/app.ts`, l'application échouera au démarrage. Cela est dû au fait que le service de routage n'est pas configuré tant que l'application n'est pas dans un état `booted`.

Fondamentalement, cette limitation impacte positivement votre code car le code de l'application devrait dépendre de la configuration, et non l'inverse.

## Mise à jour de la configuration à l'exécution

Vous pouvez modifier les valeurs de configuration à l'exécution en utilisant le service de configuration. La méthode `config.set` met à jour la valeur en mémoire, et aucun changement n'est apporté aux fichiers sur le disque.

:::note

La configuration est modifiée pour l'ensemble de l'application, pas seulement pour une seule requête HTTP. Cela est dû au fait que Node.js n'est pas un environnement d'exécution multithread, et la mémoire dans Node.js est partagée entre plusieurs requêtes HTTP.

:::

```ts
import env from '#start/env'
import config from '@adonisjs/core/services/config'

const HOST = env.get('HOST')
const PORT = env.get('PORT')

config.set('app.appUrl', `http://${HOST}:${PORT}`)
```
