# workshop-asynchronous-vue3

## Définition d'Asynchrone 

is an adjective describing objects or events that are not coordinated in time.

c'est quoi?

On dit d'un évènement qu'il est asynchrone si l'on ne peut pas prévoir son temps d'execution exactement.

## Exemples de la vie

- Un trajet en voiture (dépend de la voiture, des bouchons, de la météo, etc...)
- La cuisson d'un plat (dépend des ingrédients, de la taille des morceaux, de la  marque du four, de l'ancienneté du four, etc...)
- Un appel aux impots (dépend de l'affluence lors de l'appel, dépend du nombre d'employés affectés à répondre, dépend du bon fonctionnement du réeau téléphonique, ...)
- Une conversation (je te pose une question et je ne sais pas combien de temps tu vas mettre à réfléchir puis répondre)

## Et le dev?
En dev ce que l'on ne peut pas prévoir sont les appels à des services à travers du réseau (et d'autres trucs mais on ne va pas épiloguer dans notre cas).

Pourquoi? Le réseau n'est pas une chose fiable et ce pour plusieurs raisons : 
  - Si notre laptop est en wifi?
  - Si notre collègue est en train de télécharger qlq chose?
  - Si Sfr a décidé de faire des travaux sur la fibre de la rue?
  - Si des hackers attaquent notre réseau ?
  - Si notre server dns a crash?
  - Si le server backend est surchargé de requete par le réseau et ne peut pas répondre à tout le monde rapidement?
  
 Et ce ne sont que quelques exemples de ce qui cause de l'incertitude quant à la résolution du temps de requete sur le réseau internet.

## La route d'une requête

Maintenant il faut comprendre que chaque requête passe par énormément d'intermédiaires : 
  - Pc du dev 
  - Box wifi Recommerce
  - Server Proxy
  - Server Dns
  - Un autre server dns x4 ou 5
  - Box wifi du server
  - Server back recommerce.
  - temps de traitement du server

Et ca ce n'est que l'allée de la requete mais le back doit encore nous répondre. Soit en erreur soit avec les données demandées. Il va donc envoyer une Réponse qui devra faire entièrement le chemin  inverse et parfois même un peu plus.
Chacun des intermédiaires possède son lot d'inprédictabilité comme nous l'avons vu avant. 

Il est donc impossible de prévoir combien de temps va mettre une requete à un server, il y a trop de paramètres inconnus ou sur lesquels nous ne pouvons pas avoir de controle.

## Quel problème ça pose?

Le but d'un site web est d'afficher de la donnée et d'en récolter.
Ces deux tâches nécessitent des envois/réceptions de requêtes.

Lorsqu'un utilisateur va arriver sur la fiche d'un produit par exemple, nous allons envoyer une requete permettant de connaitre les informations du produit demandé (prix, nom, marque, date de création, distributeur, etc...). Nous allons donc contacter une API.

## C'est quoi une API ou Application Programming Interface?

C'est une interface que le server va offrir pour exposer ses données.

### Comment intérroger une API? 

Les API web que nous utilisons sont disponibles via des URLs comme par exemple : 
- https://adresse-de-mon-api.com/produit/:id

Si l'on envoie une requête sur cette url en remplacant `:id` par un nombre, selon le type de requete le server fera une opération différente : 
- GET -> Nous retourne le produit correspondant à l'ID
- PUT -> Remplace le produit correspondant à l'ID par un nouveau spécifié dans le body de notre Requete
- PATCH -> met à jour le produit correspondant à l'ID avec les informations spécifiées dans le body
- POST sur l'url https://adresse-de-mon-api.com/produit -> crée un nouveau produit avec les informations spécifiées dans le body
- ...

Chacune de ces opérations ont été créées sur le server de l'API et écoute les requetes faites sur ces urls en particuler.

Si l'on tente de contacter une API sur une URL n'existant pas ou avec les mauvais paramètres, le server nous répondra une erreur.

## Cas concret d'asynchronisme

Imaginons que l'utilisateur veut consulter une ficher produit. Notre site vient de contacter le server sur son API en lui demandant la fiche du produit ayant l'id 23.

### Que doit-il se passer?
Que se passe-t-il sur notre site pendant que la requete voyage jusqu'au backend et est traité par le server?
- Notre site doit il rester sur une page blanche? 
- Notre site doit il rester sur la même page? 
- Si l'utilisateur clique sur un bouton, doit on faire apparaitre la modale correspondante ou bien attend on que la requete se résolve avant de la faire apparaitre?

On veut evidemment que le site continue de fonctionner.
On doit Donc pouvoir faire tourner du code pendant que la requete est en train d'être résolue par le réseau/back. \
Or et c'est important de s'en souvenir, Javascript est un langage dit single threaded ce qui veut dire qu'il ne peut executer qu'une ligne à la fois. Il ne peut donc pas en même temps gérer la résolution d'une requete et l'affichage  d'une modale


Nous avons donc besoin d'un object javascript indispensable: Les promesses ou Promise.

## L'objet Promise 

The Promise object represents the eventual completion (or failure) of an asynchronous operation and its resulting value. (https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

Une promesse est un object représentant un appel asynchrone. Elle peut être dans 4 états différents : 
- 'pending' = en attente 
- 'rejected' = la réponse est en erreur
- 'fulfilled' = la réponse est validé
- 'settled' = la réponse est arrivé peut importe si elle est en erreur ou validé.

Après la création d'une Promesse en attendant qu'elle soit résolue, le thread javascript va continuer à exectuer les lignes suivantes. **ON NE RESTE PAS A ATTENDRE LA RESOLUTION.**

Nous souhaiterions pourtant faire tourner du code **seulement** une fois que la promesse est résolue. Il va nous falloir pour ça utiliser une des méthodes de Promise : `.then()`

## La méthode `.then()`
Cette méthode prend en paramètre une callback en cas de succes et une callback en cas d'erreur (optionnelle). \
La méthode then ne sera appelée que lorsque l'état de la promesse passe de pending vers rejected/fulfilled/settled.

### Qu'est ce que ca implique? 
On va pouvoir fournir à la méthode then une callback qui s'exectuera seulement lorsque la promesse sera résolue et que les données renvoyées par le back seront disponibles.

### Exemple
```
function isEven(num) {
  return new Promise((resolve, reject) => {
    if (num % 2 === 0) {
      resolve(true);
    } else {
      reject(false);
    }
  });
}
```
Cette fonction permet de déterminer si un param est pair ou impair

```
isEven(2)
  .then((result) => {
    console.log(result); // affiche "VRAI"
  })
  .catch((result) => {
    console.log(result); // ne s'exécute pas
  });

isEven(3)
  .then((result) => {
    console.log(result); // ne s'exécute pas
  })
  .catch((result) => {
    console.log(result); // affiche "FAUX"
  });
```
Nous avons ici un exemple qui montre l'utilisation de `.then()` pour attendre la résolution de la promesse avant traitement

### Usecases

Si l'on revient sur notre cas de Produit.
Si l'on crée une promesse lorsque l'on crée notre requete afin d'avoir les informations produit, on va pouvoir fournir une callback traitant la donnée seulement lorsque le back répondra. Une fois la promesse crée, notre script continuera de tourner et donc l'utilisateur pourra ouvrir sa modale en cliquant sur son bouton, le site continuera de vivre et une fois que la donnée reviendra du back, notre callback va pouvoir la traiter et remplir les champs du site qui vont bien.

Avec notre exemple le use case donne : 

- Affichage du site
- Envoie de requete
- Clique utilisateur sur le bouton
- Affichage modale
- Resolution de la requete
- affichage des informations produit

mais de part la nature asynchrone de la requete nous pourrions tout aussi bien avoir : 
- Affichage du site
- Envoie de requete
- Clique utilisateur sur le bouton
- Resolution de la requete
- Affichage modale
- affichage des informations produit

ou bien : 
- Affichage du site
- Envoie de requete
- Resolution de la requete
- affichage des informations produit
- Clique utilisateur sur le bouton
- Affichage modale

On voit bien ici qu'on ne peut pas prévoir dans quel ordre les actions vont s'effectuer. 

## Asynchronisme²

Ajoutons maintenant un peu de complexité, imaginons que l'utilisateur veut consulter un tableau de produit. \
Chaque produit se trouve sous la forme : 
```
{
  id: string,
  price: number
  name: string
}
```

Problème : les données ne sont pas stockées sur les mêmes API.
- L'api Produit contient les ID et les Name.
- L'api Pricing contient le prix associé à chaque id.

Afin de remplir notre tableau, le site va donc devoir faire plusieurs requêtes : 
- demander la listes des produits à l'api Produit
- demander pour chaque produit son prix à l'api Pricing.

Si notre tableau contient 4 produit il faudra donc faire : 
```
1 (get la liste des produits) +
4 (un appel par produit pour avoir son prix) 
= 5 appels API
```

### Question 
Si je run le code suivant : 
```
let maListDeProduit = getListProduitAsync(): Promise<Produit[]>
let produit1: Produit = maListDeProduit[0]
```
#### Que se passe-t-il?
#### Pourquoi? 
### Explication :

Pour deux raisons :
  - Car le type de maListDeProduit est Promise<Product[]>, ce n'est donc pas un tableau dans lequel je peux choper de la donnée, la deuxieme ligne plante
  - Car j'ai utilisé le resultat de l'appel getListProduit() sans m'etre assuré que la requete était belle et bien résolue.

Comment résoudre le souci ?

Nous allons utiliser la méthode `.then()`

```
let maListDeProduit = [] as Produit[]

let maCallBack = (resultatDeLaRequete: Produit[]) => {
    maListDeProduit = resultatDeLaRequete
}

let maRequete : Promise<Produit[]> = getListProduitAsync(): Promise<Produit[]>

maRequete.then(maCallBack)
```

Dans ce cas, la requete est envoyée et lorsqu'elle sera résolue maListDeProduit prendra la bonne valeur soit un tableau de produit. \
On peut simplifier ce code en transformant la call back en fonction anonyme

```
let maListDeProduit = [] as Produit[]

let maRequete : Promise<Produit[]> = getListProduitAsync(): Promise<Produit[]>

maRequete.then((resultatDeLaRequete: Produit[]) => {
    maListDeProduit = resultatDeLaRequete
})
```

On peut encore simplifier en ne créant pas de variable temporaire maRequete

```
let maListDeProduit = [] as Produit[]

getListProduitAsync().then((resultatDeLaRequete: Produit[]) => {
    maListDeProduit = resultatDeLaRequete
})
```
Le code que nous venons d'écrire va appeler l'API et ne mettra à jour la valeur de  `maListDeProduit` que lorsque le résultat sera reçu.
Nous venons de traiter un appel asynchrone de la bonne façon.

Nous souhaitons maintenant obtenir le prix de chacun des produits. Il va donc falloir contacter une nouvelle api pour chaque produit une fois que la list de produit aura été récupérée.

```
let maListDeProduit = [] as Produit[]

let prix1 : number | null = null 
let prix2 : number | null = null 
let prix3 : number | null = null 
let prix0 : number | null = null 

// on requete la list de produit
let maListDeProduit = getListProduitAsync()
// on ajoute le traitement à faire une fois la liste obtenue
.then((resultatDeLaRequete: Produit[]) => {
    // on set la valeur dans notre variable
    maListDeProduit = resultatDeLaRequete
})

// on requete les prix correspondants aux produits
getPrice(maListProduit[0]).then((result) => prix0 = result) 
getPrice(maListProduit[1]).then((result) => prix1 = result)
getPrice(maListProduit[2]).then((result) => prix2 = result)
getPrice(maListProduit[3]).then((result) => prix3 = result)

```

### Que va-ton obtenir en runnant ce code?

Encore une erreur ! On peut voir ici que nous allons mettre de la donnée dans maListDeProduit seulement lorsque la promesse sera résolue. Le fonctionnement singlethreaded de Javascript va continuer après la création de la promesse.
Il va donc appeler `getPrix()` avec comme paramètre `maListProduit[0]`. \
Pourtant `maListProduit` n'a toujours pas de donnée à l'index 0 puisque nous sommes juste apres la création de la promesse de `getListProduitAsync()`. \
Il va donc falloir attendre que `getListProduitAsync()` soit terminée ainsi que la callback passée en param.

### Comme par hasard?

La méthode `.then()` renvoie un objet Promise, nous pouvons donc enchainer un deuxième `.then()` sur le premier

```
let maListDeProduit = [] as Produit[]

let prix1 : number | null = null 
let prix2 : number | null = null 
let prix3 : number | null = null 
let prix0 : number | null = null 

// on requete la list de produit
let maListDeProduit = getListProduitAsync()
// on ajoute le traitement à faire une fois la liste obtenue
.then((resultatDeLaRequete: Produit[]) => {
    // on set la valeur dans notre variable
    maListDeProduit = resultatDeLaRequete
})
// Une fois que maListDeProduit possède de la donnée on fais un nouveau traitement
.then(() => {
    // on requete les prix correspondants aux produits
    getPrice(maListProduit[0]).then((result) => prix0 = result) 
    getPrice(maListProduit[1]).then((result) => prix1 = result)
    getPrice(maListProduit[2]).then((result) => prix2 = result)
    getPrice(maListProduit[3]).then((result) => prix3 = result)
})
```
En enchainant les `.then()` nous nous sommes assurés que les envois de requete sur l'api Pricing ont été faits une fois que l'appel sur l'api Produit a été résolu.
Cette technique s'appelle le chainage de promesse ou Promise chaining (https://javascript.info/promise-chaining)

On peut ajouter un petit sucre syntaxique afin d'attendre la résolution de toutes les promesses de getPrice à l'aide de la méthode `Promise.all()`. \
Cette méthode prend un tableau de promesse en entrée et retourne une Promesse qui se résoudra une fois que toutes celle du tableau sont résolues. Elle met à disposition le résultat des promesses dans un tableau, l'odre du tableau est conservé.

exemple : 

```
Promise.all([
   getPrice(0), 
   getPrice(1), 
   getPrice(2), 
   getPrice(3), 
])
.then((arrResultat) => {
    console.log(arrResultat[0]); // résultat de getPrice(0)
    console.log(arrResultat[1]); // résultat de getPrice(1)
    console.log(arrResultat[2]); // résultat de getPrice(2)
    console.log(arrResultat[3]); // résultat de getPrice(3)
})
```

Notre code utilisant cette méthode : 
```
let maListDeProduit = [] as Produit[]

let prix1 : number | null = null 
let prix2 : number | null = null 
let prix3 : number | null = null 
let prix0 : number | null = null 

// on requete la list de produit
let maListDeProduit = getListProduitAsync()
// on ajoute le traitement à faire une fois la liste obtenue
.then((resultatDeLaRequete: Produit[]) => {
    // on set la valeur dans notre variable
    maListDeProduit = resultatDeLaRequete
})
// Une fois que maListDeProduit possède de la donnée on fais un nouveau traitement
.then(() => {
    // on requete les prix correspondants aux produits
    Promise.all([
        getPrice(maListProduit[0]), 
        getPrice(maListProduit[1]), 
        getPrice(maListProduit[2]), 
        getPrice(maListProduit[3]), 
    ])
    .then(prices => {
        prix0 = prices[0]
        prix2 = prices[1]
        prix3 = prices[2]
        prix4 = prices[3]
    })
})
```
Les avantages de Promise.all :
 - Les requetes sont lancées en parallèle
 - Toutes les requetes sans exception doivent être résolues avant de passer dans `.then()`
 - Ca évite d'avoir à créer des boilerplates de .then pour chaque promesse
 - Si une des promesses fail alors `Promise.all()` fail aussi

 Si l'on souhaite un comportement où même si une des promesses fail les autres renvoient leur résultat et les rends disponibles il faut regarder du côté de `Promise.settle()` 
## Bonus Async/Await

Il existe une autre façon de traiter l'asynchronisme, on peut utiliser les mots `async` et `await`. \

### Qu'est ce que c'est

Async await on été introduits "récemment" pour éviter les boilerplates de .then().
Cette api permet de "simplifier" la lecture d'un fichier faisant des appels asynchrones en supprimant le besoin de callback.

### exemple

notre code avec le `.then()`
```
let maListDeProduit = [] as Produit[]

getListProduitAsync()
.then((resultatDeLaRequete: Produit[]) => maListDeProduit = resultatDeLaRequete)
```
le code avec async/await
```
let maListDeProduit : Produit[] = await getListProduitAsync()
```
il faut pas oublier d'ajouter le mot `async` devant la fonction dans laquelle se trouve le `await`

```
maFunc = async () => {
    let maListDeProduit : Produit[] = await getListProduitAsync()
}

```
Notre code sera mit en pause le temps de résoudre `getListProduitAsync()`.

## /!\ Disclaimer /!\

Async Await utilise les promesses, ce n'est que du sucre syntaxique. Il se cache derrière les comportements asynchrones avec des `.then()`

**Il ne faut pas penser que votre code sera synchrone en utilisant await async et que vous pourrez controler le temps d'exécution des méthodes** \
**Async await imite seulement un comportement synchrone**

## Erreurs courantes

Si les concepts d'asynchronisme, de Promise et de .then sont compris il reste quelques écueils dans lesquels il faut éviter de tomber.

### Les variables partagées

```
let prixCourant: number = -1
const p1 = getPrice(23).then(prix => prixCourant = prix) //getPrice retourne 1

const p2 = getPrice(50).then(prix => prixCourant = prix) //getPrice retourne 100000
.then(faire_un_virement_sur_le_compte_en_banque_de_PO(prixCourant))

```

Quelle est la valeur transférée sur le compte de PO?

Il est impossible de le savoir puisqui'l existe plusieurs possibilités : 
- 1
- 100000

Nos deux callbacks vont mettre à jour une variable partagée et vu qu'il n'est pas possible de prévoir l'ordre de résolution des Promesses on ne peut pas savoir quelle sera la valeur de cette variable. Nous pourrions arriver à l'appel de `faire_un_virement_sur_le_compte_en_banque_de_PO` tandis que P1 se résout et donc la valeur dans la variable ne sera pas celle attendue.

Un autre cas serait par exemple la construction d'un object ayant des propriétés dépendantes de plusieurs apis. Nous aurions donc plusieurs promesses se résolvant dans un ordre inconnus et notre objet pourrait contenir des valeurs érronées.

Pour éviter ce problème, il faut s'assurer soit de ne pas utiliser de variable partagée soit que l'ordre de résolution est prévisible.

Pas de variable partagée :
```
getPrice(23).then(prix => {   //getPrice retourne 1
    faire_un_virement_sur_le_compte_en_banque_de_PO(prix)
}) 

getPrice(50).then(prix => {  //getPrice retourne 100000
    faire_un_virement_sur_le_compte_en_banque_de_PO(prix)
})
```

Nous voyons ici qu'un virement sera de 1 et l'autre beaucoup trop ! Il serait possible de prévoir aussi l'ordre en chainant les promesses entre elles. Nous pouvons prévoir les comportements en supprimant les variables partagées.

Si l'on souhaite les garder :

```
let prixCourant: number = -1
const p1 = await getPrice(23) //getPrice retourne 1
faire_un_virement_sur_le_compte_en_banque_de_PO(prixCourant)

const p2 = await getPrice(50) //getPrice retourne 100000
faire_un_virement_sur_le_compte_en_banque_de_PO(prixCourant)

```
Ici nous pouvons prévoir l'ordre et le montant des virements qui seront de 1 puis beaucoup trop !

### Les loops de Promise

Voici un tableau 
```
let users = ["Ana", "Violette", "Jean", "Marc"]
``` 
Nous souhaiterions obtenir des datas sur ces utilisateurs avec la méthode 
```
const fetchData = user => {
    return fetch(`https://api.github.com/users/${user}`);
}
```

Si j'utilise une boucle pour lancer mes requetes 
```
for (let i = 0; i < users.length; i++) {
    const response = fetchData(users[i]);
    response.then(response => {
        response.json().then(user => {
            console.log(`${user.name}`);
    });
}
```

Je ne peux pas prévoir l'ordre de résolution, les consoles logs seront dans un ordre potentiellement érroné !
```
Jean
Ana
Violette
Marc
```

Nous pouvons résoudre ce problème avec async await en faisant : 

```
for (let i = 0; i < users.length; i++) {
    const response = await fetchData(users[i]);
    response.json().then(user => {
        console.log(`${user.name}`);
    })
}
```
Seulement dans ce cas les requetes ne seront pas parallèlisées et donc il y a une (grosse) perte de performance.

Ce que nous souhaitons c'est envoyer les requetes en même temps mais pouvoir prévoir l'ordre des consoles logs.

Si ca vous rappelle quelque chose c'est normal puisqu'il faut utiliser `Promise.all()` ou `Promise.settle()`

```
    const responses = await Promise.all(users.map(user => fetchData(user)));
    const data = await Promise.all(responses.map(response => response.json()));
    console.log(data);
    data.map(user => {
        console.log(`${user.name}`);
    });
```

Ici l'ordre `["Ana", "Violette", "Jean", "Marc"]` va être respecté par le console log et pourtant nous avons pu rendre nos requetes parallèle et donc diviser par 4 (taille du tableau) le temps de résolution
