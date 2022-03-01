# Tour Of Heroes Backend -
## Création d'un CRUD côté Back .

# 1
Initialisation du projet avec [Spring Initializr].
Dépendances à ajouter:

- Spring Web
- JPA
- MySQL

Obtention d'un fichier zip à decompresser dans son dossier de travail. on peut ainsi
ouvrir le dossier avec son **IDE**.



# 2
le travail se fera essentiellement dans le dossier:
**<dossier de travail>/src/main/java**
C'est là que nous allons organiser nos classes dans different packages, à la racine de celui défini lors de la création du projet.

Comment organiser notre projet?
>nous allons utiliser l' architecture [MVC]. C'est la structure idéale pour ce projet, et des frameworks comme Angular ou Spring s'y conforment parfaitement.

Convention de nommage: Pour rappel, nous allons relier ce projet back au projet front créé avec angular. Son theme tournant autour des Heros, et pour rester coherent avec le front, nous allons creer notre back sous le même thème. Plus d'employés donc, mais des super-h"ros, ce qui est tout de même plus sympa, ah.

# 3- JPA, le point de départ du backend

Pour bénéficier de toute la puissance de Spring en matiére d'opérations sur les bases de données, nous allons devoir mettre en place une *interface* qui etendra la classe **JpaRepository**.
Cette manoeuvre se nomme la création d'un **Repository**. Un ensemble d'opérations offertes par JpaRepository pour manipuler les données d'une table dans une base de données.
Puisque nous allons "jouer" avec des héros, nous allons créer une classe **HerRepository** dans un package *Repository*, à la racine du package défini lors de la création du projet( package racine ).

        package <package racine>.Repository; 
        import org.springframework.data.jpa.repository.JpaRepository;
        interface EmployeeRepository extends JpaRepository<Hero, Long> {}
        
à noter: JpaRepository a besoin de 2 choses au moins pour fonctionner. Un objet, ici notre futur héros (nous allons créer la classe - et donc le type - Hero prochainement), et un id, ici de type Long. l'id est l'information première d'un element d'une base de données. JPA prend en charge les ids de maniere quasi autonome. (avec un peu de notre aide).

Il est temps de définir notre héros. Créons une classe **Hero** dans un package *Model*. et oui, notre héros, avant d'être du muscle, c'est de la donnée, et la donnée en MVC, on la range dans la catégorie Model. Notre classe aura pour l'instant que  propriétés:
- une id
- un nom
- un pouvoir

         import java.util.Objects;

        import javax.persistence.Entity;
        import javax.persistence.GeneratedValue;
        import javax.persistence.Id;

        @Entity
        class Employee {
        
          private @Id @GeneratedValue Long id;
          private String name;
          private String role;
        
          Employee() {}
        
          Employee(String name, String role) {
        
            this.name = name;
            this.role = role;
          }
        
          public Long getId() {
            return this.id;
          }
        
          public String getName() {
            return this.name;
          }
        
          public String getRole() {
            return this.role;
          }
        
          public void setId(Long id) {
            this.id = id;
          }
        
          public void setName(String name) {
            this.name = name;
          }

          public void setRole(String role) {
            this.role = role;
          }
        
          @Override
          public boolean equals(Object o) {
        
            if (this == o)
              return true;
            if (!(o instanceof Employee))
              return false;
            Employee employee = (Employee) o;
            return Objects.equals(this.id, employee.id) && Objects.equals(this.name, employee.name)
        && Objects.equals(this.role, employee.role);
              }
            
              @Override
              public int hashCode() {
                return Objects.hash(this.id, this.name, this.role);
              }
            
              @Override
              public String toString() {
                return "Employee{" + "id=" + this.id + ", name='" + this.name + '\'' + ", role='" + this.role + '\'' + '}';
            }
        }
        
Remarquons que notre classe à quelques termes un peu etranges. Ces instructions précédées de @ s'appelent des **annotations**. Placées avant ou aprés l'élément ciblé, elles informent Spring que ce sont des éléments particuliers. Spring les prend ainsi en charge et travail dessus en coulisses.

**@entity**:
**@Id @GeneratedValue** :

Sinon rien de neuf. Un constructeur ( attention toutefois, pas d'id dans le constructeur, c'est JPA qui va le gérer. ), des getters/setters, une fonction **hasCode** pour faire bonne figure et un **toString()** au cas où.


# 3 - Les données

On quitte Spring un moment pour créer notre base de donnée. Elle sera pour l'instant très basique 
Nous pouvons utiliser DBeaver pour la creer, avec un script tel que celui-ci par exemple:

        DROP TABLE IF EXISTS `hero`;
        CREATE TABLE IF NOT EXISTS `hero` (
          `id` int NOT NULL AUTO_INCREMENT,
          `nom` varchar(50) NOT NULL,
          `pouvoir` varchar(100) NOT NULL,
          PRIMARY KEY (`id`)
        ) 
        
        INSERT INTO `hero` (`id`, `nom`, `pouvoir`) VALUES
        (1, 'Mystique', 'Peut prendre l\'apparence de ta n\'importe qui),
        (2, 'Cyclope', 'Un laser sort de ses yeux'),
        (3, 'Gambit', 'Rend les objets explosifs'),
        (4, 'Diablo', 'Peut se teleporter'),
        (5, 'Rogue', 'Absorbe les pouvoirs d\'autrui'),
        (6, 'Jubilee', 'Tire des feux d\'artifices'),
        (7, 'Juggernaut', 'I\'m the juggernaut, bitch!'),
        (8, 'Pr Xavier', 'Telepathe'),
        COMMIT;
        
C'est vraiment basique pour l'instant mais on etoffera plus tard. On structure, on mets en place, on fait tourner, et apres on ajoute des données et des fonctionnalités.

# 4- Branchement de Spring à la base de données

De retour dans notre IDE. Dans le dossier **Ressources** il existe un fichier nommé *application.properties*. c'est là que nous allons configurer Spring pour lier à notre DB

    # MySQL configuration
    
    spring.datasource.url=jdbc:mysql://localhost:3306/<nom de votre base de données>
    spring.datasource.username=<votre identifiant MySQL>
    spring.datasource.password=<votre mot de pase MySQL>
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
    
spring.jpa.show-sql=true: affiche les requetes sql dans la console, partique!
spring.jpa.hibernate.ddl-auto=update: comportement de jpa vis à vis de la DB. ici on la mets à jour à chaque action
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect: dialect utilisé, histoire tout le monde parle la même langue. MySQL_Dialect où _ est votre version de MySQL.

# 5-Test

on se place sur notre classe principale pour lancer l'application. normalement, Spring nous laisse tranquille.

On va etre prêt pour mettre en place..

# 6-Le CRUD

Le deroulement d'un crud, dans le cas d'une application web comme la notre, se déroule en plusieurs etapes:

- Coté FRONT, une action envoie une requete http sous forme d'url
- On la reçoit côté BACK via le **controleur**, qui va transmetre le necessaire à un **service** qui demande au **Repository** de recuperer les données en DB.
- Les données récupéres, on refait le chemin inverse. le controleur BACK renvoie ce dont le controleur du  FRONT a besoin pour ses propres traitements, ses propres services.

Pour bien comprendre la relation entre controleur et service, on va pas coder les classes en entier d'un coup, mais voir, pour chaque action du Crud, ce qui se passe.
On peut d'ors et deja creer une classe HeroController dans un package Controller, et une classe HeroService dans son package Service. Comprenons ce qui se passe et aprés on code.

# 7- cRud

Notre base de données comporte déjà quelques infos, nous allons pouvoir aller les chercher ( le R de CRUD, Read).

Coté controleur (HeroControler), voilà ce qui va se passer niveau code:

        @GetMapping("/all")
        public ResponseEntity<List<Hero>> getAllHeroes(){
        	List<Hero> heroesList = heroService.findAllHeroes();
        	return new ResponseEntity<>(heroesList, HttpStatus.OK);
        }
        
dans l'ordre:
- l'annotaion @GetMapping('url') informe la fonction à venir qu'elle doit se declencher à reception de l'url en parametre. elle indique aussi la nature de la requete, GET (aller récuperer quelque chose).
- c'est la fonction getAllHeroes() va renvoyer un objet de type ResponseEntity qui contiendra une liste d'objets de type Hero.
- cette liste nous sera fournie par une fonction qu'on va aller appeler dans le service (HeroService)
- on donne cette liste et un status http OK en retour, emballés dans un ResponseEntity, pour le FRONT

Côté service (HeroService).

        public List<Hero> findAllHeroes(){
        		return HeroRepository.findAll();
        	}
        	
Pas grand chose ici car le service appel une fonction du Repository déja toute prete, findAll()


Notre liste de heros en DB est créee et passée ensuite dans l'autre sens jusqu'à notre controleur

# 8-cRud , la suite

qui peut le plus peut le moins, nous allons maintenant recuperer un seul heros

Côté controlleur (HeroController):


    @GetMapping("/find/{id}")
    public ResponseEntity<Hero> getHeroById(@PathVariable("id") Long id){
    	Hero hero = heroService.findHeroById(id);
    	return new ResponseEntity<>(hero, HttpStatus.OK);
    } 
    
même structure ici.
- Une annotation correspondant à une requete de type GET, associée à une URL.
la particularité de cette URL est qu'elle fait reference à une id dynamique, qui correspondra à celle de l'objet requeté côté front une id dynamique est passée entre crochets

- notre fonction va renvoyer un ResponseEntity egalement mais celui-ci ne contiendra qu'un seul objet de type Hero. Puiqu'on doit trouver le héros correspondant à son id dans la DB, on passe l'id de l'URL en parametre de notre fonction.l'annotation **@PathVariable("id")** indique que l'id est celle passée via l'URL.
- on appel la fonction du service correspondante, toujours en passant l'id et on stock son resultat dans une variable de type Hero
- celle-ci est emballée avec un status http ok dans un Response Entity


Côté service (HeroService)










    





- Import and save files from GitHub, Dropbox, Google Drive and One Drive
- Drag and drop markdown and HTML files into Dillinger
- Export documents as Markdown, HTML and PDF

Markdown is a lightweight markup language based on the formatting conventions
that people naturally use in email.
As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually- written in Markdown! To get a feel
for Markdown's syntax, type some text into the left window and
watch the results in the right.

## Tech

Dillinger uses a number of open source projects to work properly:

- [AngularJS] - HTML enhanced for web apps!
- [Ace Editor] - awesome web-based text editor
- [markdown-it] - Markdown parser done right. Fast and easy to extend.
- [Twitter Bootstrap] - great UI boilerplate for modern web apps
- [node.js] - evented I/O for the backend
- [Express] - fast node.js network app framework [@tjholowaychuk]
- [Gulp] - the streaming build system
- [Breakdance](https://breakdance.github.io/breakdance/) - HTML
to Markdown converter
- [jQuery] - duh

And of course Dillinger itself is open source with a [public repository][dill]
 on GitHub.

## Installation

Dillinger requires [Node.js](https://nodejs.org/) v10+ to run.

Install the dependencies and devDependencies and start the server.

```sh
cd dillinger
npm i
node app
```

For production environments...

```sh
npm install --production
NODE_ENV=production node app
```

## Plugins

Dillinger is currently extended with the following plugins.
Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantaneously see your updates!

Open your favorite Terminal and run these commands.

First Tab:

```sh
node app
```

Second Tab:

```sh
gulp watch
```

(optional) Third:

```sh
karma test
```

#### Building for source

For production release:

```sh
gulp build --prod
```

Generating pre-built zip archives for distribution:

```sh
gulp build dist --prod
```

## Docker

Dillinger is very easy to install and deploy in a Docker container.

By default, the Docker will expose port 8080, so change this within the
Dockerfile if necessary. When ready, simply use the Dockerfile to
build the image.

```sh
cd dillinger
docker build -t <youruser>/dillinger:${package.json.version} .
```

This will create the dillinger image and pull in the necessary dependencies.
Be sure to swap out `${package.json.version}` with the actual
version of Dillinger.

Once done, run the Docker image and map the port to whatever you wish on
your host. In this example, we simply map port 8000 of the host to
port 8080 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 8000:8080 --restart=always --cap-add=SYS_ADMIN --name=dillinger <youruser>/dillinger:${package.json.version}
```

> Note: `--capt-add=SYS-ADMIN` is required for PDF rendering.

Verify the deployment by navigating to your server address in
your preferred browser.

```sh
127.0.0.1:8000
```

## License

MIT

**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)
[MVC]: <https://fr.wikipedia.org/wiki/Mod%C3%A8le-Vue-Contr%C3%B4leur>
   [spring initializr]: <https://start.spring.io/>
   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]: <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
   [PlMe]: <https://github.com/joemccann/dillinger/tree/master/plugins/medium/README.md>
   [PlGa]: <https://github.com/RahulHP/dillinger/blob/master/plugins/googleanalytics/README.md>
