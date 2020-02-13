# WorkshopGo

**Pour toutes questions de syntaxe : [A Tour of Go](https://tour.golang.org/)**  

****
## Installer Go

Sous Fedora :
```bash
sudo dnf install golang
```
Sous MacOS :
```bash
brew install go
```

Pour les autres distributions, référez-vous à votre package manager ou suivez directement le [guide officiel](https://golang.org/doc/install)


Assurez-vous ensuite que vous disposez d'une version à jour de go :
```bash
❯ go version
go version go1.13.8 darwin/amd64 # 🎉
```


****
## Créer un nouveau projet 

```bash
go mod init nomDuProjet
```

Vous pouvez maintenant créer un fichier `main.go` :
```Go
package main

import (
	"fmt"
)

func main() {
	fmt.Println("Hello World")
}
```

> *Qu'est ce que l'on vient de faire ?*

On vient tout simplement d'importer le package [`Fmt`]((https://golang.org/pkg/fmt/)) qui est un package vous permettant principalement d'afficher / formatter du texte. Il contient de nombreuses fonctions que vous pouvez voir dans la [doc](https://golang.org/pkg/fmt/) ou directement dans votre IDE si vous avez l'auto-complétion d'activée.

Par la suite on a simplement appelé la fonction [`Println`](https://golang.org/pkg/fmt/#Println) qui nous permet d'afficher du texte suivi d'un retour à la ligne.

<br>

> *Comment compiler mon code ?*

Un exécutable go possède obligatoirement un package `main` et une fonction `main` qui sera le point d'entrée de votre code.

Vous avez deux possibilités pour éxecuter votre code en go.

Via un `go build` pour créer un éxecutable :
```bash
> go build -o monProjet main.go
> ./monProjet
Hello World
```

Ou via `go run` pour compiler et éxecuter directement le code :
```bash
> go run main.go
Hello World
```

> *Il y a une norme de code à suivre ?*

En go vous êtes libre de coder comme bon vous semble, on ne vous tapera pas sur les doigts si vous utilisez des espaces ou des tabs comme indentation.

La seule règle à respecter est d'utiliser l'outil [gofmt](https://golang.org/cmd/gofmt/) pour formatter votre code avant de le publier.

```bash
> cat main.go # un code très moche...
package main
import "fmt"

func main()         {
fmt.Println(       "Hello World")
}

> gofmt -w main.go # on formate notre code
> cat main.go
package main

import "fmt"

func main() {
	fmt.Println("Hello World")
}
```


On vous laisse quelques détails [ici](https://blog.golang.org/go-fmt-your-code).

![package](https://github.com/egonelbre/gophers/blob/master/vector/fairy-tale/witch-learning.svg)


****
## Exercice 1 - Fondamentaux

Pour prendre en main la syntaxe du go, on va commencer par un exercice basique : print tous les chiffres primaires de 0 à 100.  
Pour rappel, un chiffre primaire est un entier positif qui est divisible seulement par 1 et lui-même (1 exclu).

Le résultat doit ressembler à ca :
```
2 3 5 7 11 13 17 19 23 29 31 37 41 43 47 53 59 61 67 71 73 79 83 89 97
```

On vous demande bien sur de ne pas utiliser une fonction déjà existante mais de créer la vôtre.

![Climb](https://github.com/egonelbre/gophers/blob/master/vector/adventure/hiking.svg)


****
## Exercice 2 - Lister les fichiers

Maintenant que vous connaissez les bases, nous pouvons commencer le projet du workshop.  
La première étape va être de créer une fonction capable de lister les fichiers et les dossiers.

Je vous conseil d'utiliser la fonction `ReadDir` du paquet `io/ioutil` pour récupérer les fichiers et dossier contenus dans un dossier.

![read](https://github.com/egonelbre/gophers/blob/master/sketch/fairy-tale/messenger-reading.png)


****
## Exercice 3 - Première route d'API

Une API (Application Programming Interface) web permet de faire des requêtes sur un url pour donner ou obtenir des informations depuis un service.

Pour la première route de notre API, nous allons faire un simple hello world.

Le paquet `net/http` de Go nous permet de créer des routes très simplement et sans avoir besoin de librairie externe.  
Tout d'abord, il nous faut écouter sur un port afin de recevoir des requêtes.  
```Go
log.Fatal(http.ListenAndServe(":8080", nil))
```

La fonction qui nous intéresse est `HandleFunc`. Elle nous permet d'assigner une route à une fonction.

```Go
http.HandleFunc("/hello", helloWorld)
```

Le premier paramètre est donc le nom de la route et le deuxième est la fonction qui va être appelée lorsque quelqu'un va appeler la route en question.
Une fonction de route doit se présenter sous cette forme :

```Go
func helloWorld(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")      // réponse de type JSON
	w.WriteHeader(http.StatusOK)                            // code de réponse HTTP                      
	w.Write([]byte(`{"message": "hello world"}`))           // envoie de la réponse
}
```
Une fonction de route prend toujours ces deux paramètres. Le premier (`w http.ResponseWriter`) nous permet de renvoyer une réponse et le deuxième (`r *http.Request`) nous permet de récupérer les informations contenues dans la requête.

**Essayer d'utiliser ce code et de rajouter une route pour bien comprendre le fonctionement**

Code complet : 
```Go
import (
	"log"
	"net/http"
)

func helloWorld(w http.ResponseWriter, r *http.Request) {
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)
	w.Write([]byte(`{"message": "hello world"}`))
}

func main() {
	http.HandleFunc("/hello", helloWorld)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```
Pour tester les routes, un simple curl suffit.  
```bash
$ curl localhost:8080/hello
{"message": "hello world"}
```

![api](https://github.com/ashleymcnamara/gophers/blob/master/MovingGopher.png)


****
## Exercice 4 - Renvoyer la liste de fichiers

Pour combiner l'exercice 2 et 3, nous allons renvoyer la liste de fichiers récupérés dans l'exercice 2 sur une nouvelle route au format JSON.  
Vous allez devoir transformer un tableau de string en JSON en utilisant la fonction `json.Marshal` du paquet `encoding/json`.

Résultat attendu :
```bash
$ curl localhost:8080/list
["go.mod","main.go", ...]
```


****
## Exercice 5 - Prendre un paramètre dans la requête

Nous allons maintenant envoyer un paramètre à la requête de l'api pour spécifier le dossier ou lister les fichiers.

Pour récupérer des paramètres au format json, il faut  d'abord définir notre structure de donnée à récupérer.
```Go
type listParameters struct {
	Path string
}
```

Il nous suffit ensuite de créer un nouveau décodeur de JSON puis de décoder les paramètres en utilisant notre structure.

```Go
decoder := json.NewDecoder(r.Body)		// Création du décodeur
var data listParameters					// Décalaration la variable local du type de la structure
err := decoder.Decode(&data)			// Décodage du json dans notre structure
if err != nil {							// Vérification des erreurs
	log.Fatal(err)
}
```

On peut ensuite utiliser notre variable data de cette manière : `data.Path`

Résultat attendu :
```bash
$ curl localhost:8080/list --data '{"path": "."}'
["go.mod","main.go", ...]
```


****
## Exercice 6 - Download un fichier

Créez une nouvelle route qui prend un path en paramètre et qui permet de télécharger le fichier correspondant au path donné.
Pour permettre le téléchargement d'un fichier, on peut simplement utiliser la fonction `http.ServeFile()`.

Résultat attendu :
```bash
$ curl localhost:8080/dl --data '{"path": "/tmp/gopher"}'
random file content
```

```bash
$ curl localhost:8080/dl --data '{"path": "/tmp/gopher"}' -o gopher
$ ls
gopher
$ cat gopher
random file content
```

![dl](https://github.com/ashleymcnamara/gophers/blob/master/LazyGopher.png)


****
## Exercice 7 - Upload un fichier

Créez une nouvelle route permettant d'upload un fichier.  
Vous ne devez plus envoyer un json en paramètre, mais un fichier.

Pour le reste, on vous laisse chercher par vous-même. Hésitez pas à nous poser des questions si besoin.

![go10](https://blog.golang.org/10years/gopher10th-small.jpg)
