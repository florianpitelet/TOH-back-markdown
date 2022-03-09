# Tour Of Heroes Backend -
## Création d'un CRUD côté Back .

---

## Table des matières
- [1-Preparation de l'environnement de travail](#1)
- [2-Organisation du projet](#2)
- [3-JPA, le point de départ du backend](#3)
- [4-Les données](#4)
- [5-Branchement de Spring à la base de données](#5)
- [6-Test](#6)
- [7-Le CRUD](#7)
- [8-Le contrôleur (HeroControler)](#8)
- [9-c**R**ud](#9)
- [10-c**R**ud , la suite](#10)
- [11- cru**D**, Delete](#11)
- [12-**C**rud, Create](#12)
- [13-cr**U**d, update](#13)
- [14-Premier bilan](#14)
- [15-Quand ça marche c'est bien mais quand ça marche pas?](#15)
- [16-Conclusion?](#16)
- [17-Attention ça se "**CORS**"](#17)
- [18-Aller plus loin](#18)
- [19-JPA à la rescousse!](#19)
- [20-Mise à jour des entités Team et Hero](#20)
- [21-Ajouter/Enlever un héros d'une équipe](#21)

---

### 1-Preparation de l'environement de travail{#1}
Initialisation du projet avec [Spring Initializr].
Dépendances à ajouter:

- Spring Web
- JPA
- MySQL

Obtention d'un fichier zip à décompresser dans son dossier de travail. On peut ainsi
ouvrir le dossier avec son **IDE**.



### 2-Organisation du projet{#2}
Le travail se fera essentiellement dans le dossier :
**<dossier de travail>/src/main/java**
C'est là que nous allons organiser nos classes dans diffèrent packages, à la racine de celui défini lors de la création du projet.

Comment organiser notre projet?
>nous allons utiliser l' architecture [MVC]. C'est la structure idéale pour ce projet, et des frameworks comme Angular ou Spring s'y conforment parfaitement.

Convention de nommage : Pour rappel, nous allons relier ce projet back au projet front créé avec angular. Son thème tournant autour des Héros, et pour rester cohérent avec le front, nous allons créer notre back sous le même thème. Plus d'employés donc, mais des super-héros, ce qui est tout de même plus sympa, ah.

### 3-JPA, le point de départ du backend{#3}

Pour bénéficier de toute la puissance de Spring en matière d'opérations sur les bases de données, nous allons devoir mettre en place une *interface* qui étendra la classe **JpaRepository**.
Cette manœuvre se nomme la création d'un **Repository**. Un ensemble d'opérations offertes par JpaRepository pour manipuler les données d'une table dans une base de données.
Puisque nous allons "jouer" avec des héros, nous allons créer une classe **HeroRepository** dans un package *Repository*, à la racine du package défini lors de la création du projet (package racine).

        Package <package racine>. Repository ; 
        import org.springframework.data.jpa.repository.JpaRepository;
        interface EmployeeRepository extends JpaRepository<Hero, Long> {}
        
à noter : JpaRepository a besoin de 2 choses au moins pour fonctionner. Un objet, ici notre futur héros (nous allons créer la classe - et donc le type - Hero prochainement), et un id, ici de type Long. l'id est l'information première d'un élément d'une base de données. JPA prend en charge les ids de manière quasi autonome. (Avec un peu de notre aide).

Il est temps de définir notre héros. Créons une classe **Hero** dans un package *Model*. Et oui, notre héros, avant d'être du muscle, c'est de la donnée, et la donnée en MVC, on la range dans la catégorie Model. Notre classe aura pour l'instant que propriétés :
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

- **@entity**: Annonce à JPA que la classe est une entité. Cela se traduit par la création d'une table en Base de données, si celle-ci n'existe pas déjà.
- **@Id @GeneratedValue** : On declare le champ suivant l'annotation comme étant une Id générée, gérée et incrémentée automatiquement.

C'est une des forces de JPA, qui va modifier la DB liée au projet via le code. Pour être précis, un module dédié, Hibernate en l'occurence, qui va se charger d'interpreter ces annotations et de générer les requêtes SQL.

Sinon rien de neuf. Un constructeur (attention toutefois, pas d'id dans le constructeur, c'est JPA qui va le gérer.), des getters/setters, une fonction **hasCode** pour faire bonne figure et un **toString()** au cas où.


### 4-Les données{#4}

On quitte Spring un moment pour créer notre base de donnée. Elle sera pour l'instant très basique 
Nous pouvons utiliser DBeaver pour la créer, avec un script tel que celui-ci par exemple :

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
        (7, 'Juggernaut', 'I\'m the juggernaut, b***h!'),
        (8, 'Pr Xavier', 'Télépathe'),
        COMMIT;
        
C'est vraiment basique pour l'instant mais on étoffera plus tard. On structure, on met en place, on fait tourner, et après on ajoute des données et des fonctionnalités.

### 5-Branchement de Spring à la base de données{#5}

De retour dans notre IDE. Dans le dossier **Ressources** il existe un fichier nommé *application.properties*. C’est là que nous allons configurer Spring pour lier à notre DB

    # MySQL configuration
    
    spring.datasource.url=jdbc:mysql://localhost:3306/<nom de votre base de données>
    spring.datasource.username=<votre identifiant MySQL>
    spring.datasource.password=<votre mot de passe MySQL>
    spring.jpa.show-sql=true
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
    
spring.jpa.show-sql=true: affiche les requêtes SQL dans la console, pratique!
spring.jpa.hibernate.ddl-auto=update : comportement de jpa vis à vis de la DB. Ici on la mets à jour à chaque action
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect: dialect utilisé, histoire tout le monde parle la même langue. MySQL_Dialect où _ est votre version de MySQL.

### 6-Test{#6}

On se place sur notre classe principale pour lancer l'application. Normalement, Spring nous laisse tranquille.

On va être prêt pour mettre en place...

### 7-Le CRUD{#7}

Le déroulement d'un crud, dans le cas d'une application web comme la nôtre, se déroule en plusieurs étapes :

- Coté FRONT, une action envoie une requête http sous forme d'url.
- On la reçoit côté BACK via le **contrôleur**, qui va transmettre le nécessaire à un **service**. Le service va implémenter les fonctions d'une Interface **HeroRepository**, qui étend elle-même la classe **JpaRepositoty**..
- HeroRepository bénéficie donc des fonctions toutes prêtes de JPA en rapport avec les manipulations dans la DB. Et comme toute interface, elle peut déclarer ses propres méthodes.
- Les données récupérées, on refait le chemin inverse. Le contrôleur BACK renvoie ce dont le contrôleur du FRONT a besoin pour ses propres traitements, ses propres services.




### 8-Le contrôleur (HeroControler){#8}

Le contrôleur propose les points d'entrées de notre Front et des requêtes que ce dernier apporte. On peut imaginer un messager, à la croisée de plusieurs chemins, qui, à la réception d'une requête, prend la bonne voie pour aller chercher le service adapté.

Notre base de données comporte déjà quelques infos, nous allons pouvoir faire le CRUD dessus, tout d'abord en allant toutes les chercher ( le R de CRUD, Read).

Coté contrôleur (HeroControler), voilà ce qui va se passer niveau code :

        @GetMapping("/all")
        public ResponseEntity<List<Hero>> getAllHeroes(){
        	List<Hero> heroesList = heroService.findAllHeroes();
        	return new ResponseEntity<>(heroesList, HttpStatus.OK);
        }
        
Dans l’ordre :
- l'annotation @GetMapping('url') informe la fonction à venir qu'elle doit se déclencher à réception de l'url en paramètre. Elle indique aussi la nature de la requête, GET (aller récupérer quelque chose).
- c'est la fonction getAllHeroes() va renvoyer un objet de type ResponseEntity qui contiendra une liste d'objets de type Hero.
- cette liste nous sera fournie par une fonction qu'on va aller appeler dans le service (HeroService)
- on donne cette liste et un status http OK en retour, emballés dans un ResponseEntity, pour le FRONT

Côté service (HeroService).

        public List<Hero> findAllHeroes(){
        		return HeroRepository.findAll();
        	}
        	
Pas grand-chose ici car le service appel une fonction du Repository déjà toute prête, findAll()


Notre liste de héros en DB est créée et passée ensuite dans l'autre sens jusqu'à notre contrôleur

### 9-cRud , la suite{#9}

Qui peut le plus peut le moins, nous allons maintenant récupérer un seul héros

Côté contrôleur (HeroController):


    @GetMapping("/find/{id}")
    public ResponseEntity<Hero> getHeroById(@PathVariable("id") Long id){
    	Hero hero = heroService.findHeroById(id);
    	return new ResponseEntity<>(hero, HttpStatus.OK);
    } 
    
Même structure ici.
- Une annotation correspondant à une requête de type GET, associée à une URL.
La particularité de cette URL est qu'elle fait référence à une id dynamique, qui correspondra à celle de l'objet requeté côté FRONT. Une id dynamique est passée entre crochets.

- notre fonction va renvoyer un ResponseEntity également mais celui-ci ne contiendra qu'un seul objet de type Hero. Puisqu’on doit trouver le héros correspondant à son id dans la DB, on passe l'id de l'URL en paramètre de notre fonction. L’annotation **@PathVariable("id")** indique que l'id est celle passée via l'URL.
- on appelle la fonction du service correspondante, toujours en passant l'id et on stock son résultat dans une variable de type Hero.
- celle-ci est emballée avec un status http ok au sein d'un Response Entity.


Côté service (HeroService) :

Idem, on appelle une fonction du service (findHeroById(id)) qui appel une fonction du Repository. La particularité dans ce cas, c'est qu'on ne fait pas appel à une fonction toute prête du Repository, mais une fonction "maison". La raison pour laquelle on fait ça est que la fonction toute prête du Repository ne sait pas qu'elle doit renvoyer un objet de type Hero. On crée donc une fonction adaptée qui renvoi un objet de type Hero qui pourra être récupérée par notre Front, à l'autre bout de la chaine. C'est une des forces de JPA, qui propose des fonctions "clé en main" mais offre aussi la possibilité de créer ses propres fonctions adaptées à ses besoins.

### 10-cRud , la suite{#10}

Modifions un élément de notre DB. Côté contrôleur (HeroControler):

    @PutMapping("/update")
    public ResponseEntity<Hero> updateHero(@RequestBody Hero hero){
      Hero updatedHero = heroService.updateHero(hero);
      return new ResponseEntity<>(updatedHero, HttpStatus.CREATED);
    }

Même punition.
- Une annotation adaptée ( ici de type PUT, avec une URL qui va bien ).
- notre fonction prend en paramètre le hero, sélectionné côté FRONT, que l'on va éditer. Notez la présence d'une nouvelle annotation **@RequestBody** qui indique que la requête amène un objet avec toutes ses propriétés, car on ne sait pas à l'avance la ou lesquelles on va modifier.
- On appelle la fonction updateHero(hero) du service qui appel la fonction toute prête du Repository, save(hero). On stock le héros modifié dans une variable locale.
- on renvoi cette variable et son contenu avec un http status OK.

### 11- cruD, Delete{#11}

Vous commencez à prendre le pli peut-être :

    @DeleteMapping("/delete/{id}")
    public ResponseEntity<Hero> deleteHero(@PathVariable("id") Long id){
      heroService.deleteHeroById(id);
      return new ResponseEntity<>(HttpStatus.OK);
    }

Comme pour le find, on trouve le héros à effacer grâce à son id, donc on la passe dynamiquement dans l'URL. Notez aussi le type de l'annotation.
Le service appel sa fonction delete qui appelle celle du Repository.

### 12-Crud, Create{#12}

Bon mais c'est plus sympa de créer des héros que de les supprimer non ? 

    @PostMapping("/add")
    public ResponseEntity<Hero> addHero(@RequestBody Hero hero){
      Hero newHero = heroService.addHero(hero);
      return new ResponseEntity<>(newHero, HttpStatus.CREATED);
    }

Une annotation de type POST, l'URL qui va bien. Le service appel son add, qui appel le save du repository. Mais pas si vite !
Car si on crée un Héros il lui faut un nom et un pouvoir, ce sont ses propriétés, et, normalement, côté FRONT on a surement rempli un formulaire avec ces infos-là.
Comment vont-elles arriver jusqu’au repository ?
Rappelez-vous, l'annotation @RequestBody nous envoie un Héros complet. Celui-ci passe dans notre service et c'est là qu'on va définir les propriétés de notre héros.

La méthode addHero(hero) dans HeroService :

    public Hero addHero(Hero hero) {
        hero.setNom(hero.getNom());
        hero.setPouvoir(hero.getPouvoir());
        
        return HeroRepository.save(hero);
      }


Que se passe-t-il ? on utilise les getters et setters de notre classe Hero (depuis le temps qu’ils ne servaient à rien ils devaient s’ennuyer) tout simplement, puis on utilise la méthode save () du repository, à laquelle on passe le nouveau héros tout frais. C'est ce nouveau héros qui sera renvoyé en FRONT.

### 13-crUd, update{#13}
Pour éditer un héros on va faire appelle à la methode save() du repository, via HeroService encore une fois. 

    @PutMapping("/update")
public ResponseEntity<Hero> updateHero(@RequestBody Hero hero){
	Hero updatedHero = heroService.updateHero(hero);
	return new ResponseEntity<>(updatedHero, HttpStatus.OK);
}

Dans ce cas on va chercher un objet Hero complet et non pas juste son Id. En effet, côté, FRONT, on peut imaginer que l'utilisateur va remplir un formulaire pre-rempli avec les infos d'un Héros selectionné. Il va ensuite mettre à jour les champs qui l'interessent. Enfin le FRONT va transmettre au BACK l'objet mofifié entier. Le Front va sauvagerder cet objet en DB. JPA reconnait l'Id du héros et donc update celui-ci au lieu de faire un ajout.

Le code côté HeroService :

    	public Hero updateHero(Hero hero) {
		return HeroRepository.save(hero);
	}


### 14- Premier bilan{#14}

On constate que JPA, avec sa classe JpaRepository nous facilite la vie avec des méthodes toute prêtes pour gérer les principales fonctions liées au CRUD. Il est aussi suffisamment souple pour nous permettre de surcharger ses fonctions ou en créer des maisons adaptées à nos besoins. C'est l'avantage énorme que nous apporte les **interfaces**.

Pourquoi avoir besoin d'un couche service ?

La couche service va faire du traitement et du contrôle de données en amont et en aval du repository. Dans le cas d'un Add, il prépare l'objet avant qu'il soit enregistré en DB. La couche service nous permet de faire de la gestion d'exception, et donc d'améliorer la robustesse et la sécurité de notre application, comme nous allons le voir dans le prochain exemple.

### 15-Quand ça marche c'est bien mais quand ça marche pas ? {#15}

Reprenons l'exemple d'un aspect du CRUD, le "find by id". On souhaite récupérer un élément en appelant son Id.

IL peut arriver :
1- que l'on recherche un élément qui n'existe pas
2- ou qui ne soit pas du bon type.
Si l'on regarde notre fonction findHeroById(id) dans HeroRepo, elle renvoi un objet de type Hero. Donc dans le premier cas, la fonction ne renverra pas un Hero vide, il y a un risque d'erreur. Dans le second cas, il y aura une erreur.

Une solution serait d'utiliser un objet **Optionnal** contenant un objet de type Hero. Comme son nom l'indique, un objet Optionnal comprendra qu'il n'est pas nécessaire d'obtenir à tout prix un objet Hero. Cela nous permet par la suite, dans la fonction findHeroById(Long id) de **HeroService**, de surveiller une éventuelle exception et donc d'ajouter un comportement adapté aux 2 cas présentés précédemment.

La nouvelle fonction findHeroById(id) dans HeroRepo :

    Optional<Hero> findHeroById(Long id);



la nouvelle fonction findHeroById(Long id) de HeroService:

    public Hero findHeroById(Long id) { return HeroRepository.findHeroById(id)
            .orElseThrow( () -> new HeroNotFoundException("Hero by id not found"));
      }

Ici à notre return on a ajouter une fonction qui va créer une instance de la classe HeroNotFoundException(message: string). Créons cette classe dès maintenant dans un package nommé **Exception**, elle est très simple :

  <package racine>.exception;

  public class HeroNotFoundException extends RuntimeException {

    public HeroNotFoundException(String message) {
      super(message);
    }
  }


### 16-Conclusion?{#16}

Voilà à quoi devrait ressembler notre projet BACK. Observez bien comment sont formées nos classe HeroControler, HeroService et HeroRepo.
(Lien vers le repo du projet back).

*HeroControler:*

Les points importants à noter. Il y a deux annotions qui précèdent la déclaration de notre classe.

@RestController: Elle indique à Spring que cette classe est un contrôleur de type REST, qui va gérer la réception d'URL et appeler les services adaptés.

@RequestMapping("/hero"): Elle va définir la racine des URL liées aux opérations CRUD. On indique au contrôleur qu'il attend les URL suivantes :

http://localhost:8080/hero/...

http://localhost:8080/hero/all -> récupération de tous les héros
http://localhost:8080/hero/find/{id} ->récupération d'un héros
http://localhost:8080/hero/add -> ajout d'un héros
http://localhost:8080/hero/update -> Edition d'un héros
http://localhost:8080/hero/delet/{id} -> suppression d'un héros

On déclare un objet de type HeroService.

Dans le constructeur de notre classe HeroControler nous passons en paramétre l'objet de type HeroService, que l'on va ensuite créer. L'annotation @Autowirred indique à Spring qu'il doit injecter le service dans le contrôleur.

*HeroService:*

Une annotation @Service avant la déclaration de la classe indique qu'il s'agit d'un service.
On déclare un objet de type HeroRepository.
@Autowirred avant le constructeur pour indiquer à la classe qu'on lui injecte HeroRepository.
on passe l'objet HeroRepository en paramètre du constructeur et on l'instancie.


### 17-Attention ça se "**CORS**"{#17}

Vous avez un back et front qui marche mais L'application à ce stade n'est pas encore 100%fonctionnelle. Pourquoi?

Si vous êtes aussi impatient que moi vous avez sans doute déjà lancé votre back et votre front pour tout tester, cependant il y a un pépin.
ouvrons notre front dans un navigateur internet. Affichons les outils de développement de ce navigateur. Dans l'onglet Console, nous pouvons observer ce qui va se passer.
Testons une fonctionnalité, comme un ajout de héros par exemple. Dans la console vous devriez voir quelque chose de ce genre s’afficher :

    Access to XMLHttpRequest at 'http://localhost:8080/hero/add' from origin 'http://localhost:4200' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.

Le problème est le suivant : Notre FRONT est situé en localhost:4200 et notre Back est situé en localhost:8080. Pour des raisons de sécurité les navigateurs internet empêchent les requêtes de passer d'un domaine à un autre. Cela évite qu'un site malveillant situé sur un domaine puisse interagir avec le site de votre banque sur situé sur autre domaine. Logique, mais cela nous empêche de travailler.

La solution : Indiquer à Spring que l'on va autoriser les échanges entre nos deux domaines. Pour cela on va injecter du code dans la classe principale de notre projet BACK.
 Alors là il va falloir me faire confiance et juste coller cette portion de code dans la classe principale :

    @Bean
	public CorsFilter corsFilter() {
		CorsConfiguration corsConfiguration = new CorsConfiguration();
		corsConfiguration.setAllowCredentials(true);
		corsConfiguration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));
		corsConfiguration.setAllowedHeaders(Arrays.asList("Origin", "Access-Control-Allow-Origin", "Content-Type",
				"Accept", "Authorization", "Origin, Accept", "X-Requested-With",
				"Access-Control-Request-Method", "Access-Control-Request-Headers"));
		corsConfiguration.setExposedHeaders(Arrays.asList("Origin", "Content-Type", "Accept", "Authorization",
				"Access-Control-Allow-Origin", "Access-Control-Allow-Origin", "Access-Control-Allow-Credentials"));
		corsConfiguration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
		UrlBasedCorsConfigurationSource urlBasedCorsConfigurationSource = new UrlBasedCorsConfigurationSource();
		urlBasedCorsConfigurationSource.registerCorsConfiguration("/**", corsConfiguration);
		return new CorsFilter(urlBasedCorsConfigurationSource);
	}


Notons à la ligne :

     corsConfiguration.setAllowedOrigins(Arrays.asList("http://localhost:4200"));

On met l'adresse de notre FRONT. Voilà on peut maintenant communiquer sans soucis entre FRONT et BACK.




### 18-Aller plus loin{#18}

Nous avons désormais un BACK fonctionnel, qui si tout va bien, est raccordé à notre FRONT. Que fait notre application ? Nous affichons un ensemble de héros ainsi que leurs noms et pouvoir. On peut ajouter, éditer, ou supprimer un héros.



Une fonctionnalité qui serait intéressante à implémenter serait de pouvoir créer une équipe de héros, qui aurait ses propres caractéristiques.
Comment cela pourrait se traduire côté BACK ?

Niveau DB, il faudrait ajouter une table Team avec une id en clé primaire et un nom.
Dans la table Hero, il faudra ajouter une clé étrangère liée à l'id de la table équipe.

Au niveau de notre projet nous allons créer une nouvelle entité Equipe qui aura son propre CRUD, et donc son propre contrôleur et son propre service.

### 19-JPA à la rescousse ! {#19}

La force de JPA est de pouvoir faire les deux simultanément directement à l'aide de notre code et sans passer par la case bash MySQL.

Il est important de noter le lien entre la classe de l'entité et son impact sur la base de données. Quand on crée une classe que l'on déclare ensuite à Spring comme étant une entité (avec l'annotation @Entity), JPA créé une table du nom de la classe, et des colonnes nommées selon les propriétés de la classe. Via le code, on peut aussi mettre en place les relations entre les tables, telles que One To Many, ou Many to One, etc...

Il faut donc, avant de coder, réfléchir à nos données, tables et relations.

Dans notre projet, un héros pourra appartenir à une et seulement une équipe, et une équipe pourra contenir un ou plusieurs héros. Il s'agit donc d'une relation :
- One To Many du point de vue Team
- Many To One du point de vue Hero

Pour illustrer le propos, observons une classe Parent et une classe Enfant qui vont appliquer ces préceptes.

Classe Parent (Abrégée pour une meilleure lisibilité) :

    package xxx;

    
    imports
    ...
    ...

    @Entity
    public class Parent {

    @OneToMany(mappedBy = "parent")
    private Set<Enfant> listeEnfants = new HashSet<>();

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;

    private String nom;

    //Constructeur vide
    public Parent() {
    }

    //Second constructeur
    public Parent(Long id, String nom){
        super();
        this.id = id;
        this.nom = nom;

    }


        //Getters et Setters
        // pour id
        // pour nom
        // Et aussi pour listeEnfants-> 
        
        public Set<Enfant> getListeEnfants(){
            return listeEnfants
          }
    }


Ce sont les annotations qui vont "habiller" notre classe et informer JPA de leur rôle au sein de la table qui va être créée.

- @Entity: déclare l'entité
- @OneToMany(mapedBy = ""): On met en place la relation avec une future table enfant (qu’on va décrire plus loin), en associant cette annotation avec une collection d'objets de type Enfant.
- @Id, @GeneatedValue(...), @Column se chargent de déclarer une colonne id  auto incrémentée non nullable et non modifiable.

---

La classe Enfant:

    package xxx;

    imports
    ...
    ...

    @Entity

    public class Enfant {


    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    private String nom;
    private String prénom;

    @ManyToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "parent_id", referencedColumn = "id")
    private Parent parent;

        //constructeur vide
        public Enfant() { super(); }

        //Second constructeur
        public Hero(long id, String nom, String pouvoir, Team team) {
          super();
          this.id = id;
          this.nom = nom;
          this.pouvoir = pouvoir;
          this.team = team;
    	}

        

        //Getters et Setters
        // et aussi un getter pour le parent déclaré plus haut:
            public Parent getParent() {
              return parent;
            }
    }


- Même démarche ici sauf que la relation est de type Many To One
- @JoinColumn va créer une colonne parent_id qui fera référence à la clé primaire(id) de la table parent. Il faut donc lui donner une référence à un objet de type parent.

Voici à quoi les tables devraient ressembler:

table parent:

    field     type      null      key     default     extra
    id        bigint    no        PRI     NULL        auto_increment
    nom       varchar   yes               NULL



table enfant:

    field     type      null      key     default     extra
    id        bigint    no        PRI     NULL        auto_increment
    nom       varchar   yes               NULL
		prenom    varchar   yes               NULL
    parent_id bigint    yes       MUL     NULL



# 20- Mise à jour des entités Team et Hero

Les classes Team et Hero devront donc ressembler à cela désormais.

Classe Team (équivalent de la classe Parent):
      package com.pitchup.heromanager.model;

import com.fasterxml.jackson.annotation.JsonIgnore;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.OneToMany;
import java.io.Serializable;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Entity
@Table(name= "team")
public class Team implements Serializable {



    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    private Long id;

    @Column(name = "nom")
    private String nom;

    
    @OneToMany(cascade = CascadeType.ALL)
    @JoinColumn(name = "team_id", referencedColumnName = "id")
    private List<Hero> heroList = new ArrayList<>();


    public Team() { super();}

    public Team(Long id, String nom){
        super();
        this.nom = nom;
    }

    public long getId() { return id; }
    public void setId(Long id){ this.id = id; }

    public String getNom(){ return nom; }
    public void setNom(String nom){ this.nom = nom;}

    public List<Hero> getHeroList() {
        return heroList;
    }

    public void setHeroList( List<Hero> heroList){
        this.heroList = heroList;
    }

    @Override
    public String toString() {return "Team [id=" + id + ", nom=" + nom + "]";}


}

---

Classe Hero (équivalent de la classe Enfant):
      package com.pitchup.heromanager.model;

import java.io.Serializable;

import javax.persistence.CascadeType;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.ManyToOne;

@Entity
@Table(name = "hero")
public class Hero implements Serializable {


	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(nullable = false, updatable = false)
	private long id;

	private String nom;
	private String pouvoir;


	@ManyToOne
	private Team team;



	public Hero() {
		super();

	}

	public Hero(long id, String nom, String pouvoir, Team team) {
		super();
		this.nom = nom;
		this.pouvoir = pouvoir;
		
	}

	public long getId() {
		return id;
	}

	public void setId(long id) {
		this.id = id;
	}

	public String getNom() {
		return nom;
	}

	public void setNom(String nom) {
		this.nom = nom;
	}

	public String getPouvoir() {
		return pouvoir;
	}

	public void setPouvoir(String pouvoir) {
		this.pouvoir = pouvoir;
	}


	@Override
	public String toString() {
		return "Hero [id=" + id + ", nom=" + nom + ", pouvoir=" + pouvoir + ']';
	}

}


Grace aux annotations, nos deux classes sont liées au niveau DB. Il va falloir maintenant mettre en place la logique.


# 21-Ajouter/Enlever un héros d'une équipe

Spiderman nous apprends qu'un grand héros à de grandes responsabilités. Cependant, ajouter ou enlever un héros d'une équipe est une responsabilité qui revient à l'entité Equipe, car c'est une entité mére, et les héros des entités filles.

En terme d'usage dans l'application, l'ajout/suppression d'un héros d'une équipe se fera dans la page teams.
Le TeamController s'occupera de la manipulations des données côté BACK.

Pour l'ajout il attendra une URL de type: ("team/add/*id de l'équipe*/hero/*id du héros*)
Pour la suppression il attendra une URL de type: ("team/remove/*id de la team*/hero/*id du héros*")

fonction addHeroToTeam dans TeamController:

    @PostMapping("/add/{teamId}/hero/{heroId}")
    public ResponseEntity<Team> addHeroToTeam(@PathVariable Long teamId,
                                              @PathVariable Long heroId)
    {
        Team team = teamService.addHeroToTeam(teamId, heroId);
        return new ResponseEntity<>(team, HttpStatus.OK);
    }

fonction addHeroToTeam dans TeamService:

      public Team addHeroToTeam(Long teamId, Long heroId){
        Team team = this.findTeamById(teamId);
        Hero hero = HeroService.findHeroById(heroId);

        team.addHero(hero);

        TeamRepository.save(team);

        return team;
    }
On ne fait pas de demande qui n'existe pas déja au sein de TeamRepository, on fait juste un save().

A noter, le team.addHero(hero) qui va se charger d'ajouter un héros dans le tableau de héros de la classe Team, qu'on a créé.
il faut par contre creer la méthode, trés simplement:

fonction addHero de la classe Team:

      public void addHero(Hero hero){
        this.heroList.add(hero);
      }





---

Pour la suppression c'est assez smilaire excepté que la requete est de type DELETE au lieu de POST.

fonction removeHeroFromTeam dans TeamController:

    @PostMapping("/add/{teamId}/hero/{heroId}")
    public ResponseEntity<Team> removeHeroFromTeam(@PathVariable Long teamId,
                                              @PathVariable Long heroId)
    {
        Team team = teamService.removeHeroFromTeam(teamId, heroId);
        return new ResponseEntity<>(team, HttpStatus.OK);
    }

fonction removeHeroFromTeam dans TeamService:

      public Team removeHeroFromTeam(Long teamId, Long heroId){
        Team team = this.findTeamById(teamId);
        Hero hero = HeroService.findHeroById(heroId);

        team.removeHero(hero);

        TeamRepository.save(team);

        return team;
    }

Ajoutons la méthode removeHero dans la classe Hero:


      public void removeHero(Hero hero){
        this.heroList.add(hero);
      }

Ainsi quand le FRONT enverra une requête d'ajout de héros dans une équipe, sa ligne correspondante en DB sera mise à jour, avec une id dans sa colonne team_id qui fera réference à l'id de l'équipe qui l'acceuil à présent.

