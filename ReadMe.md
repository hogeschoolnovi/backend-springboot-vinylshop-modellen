# Inleiding
Je hebt in de vorige opdracht voor `GenreEntity` een Controller, Service en Repository gemaakt.
Je hebt hetzelfde gedaan voor de `PublisherEntity`.

Heb je de opdracht van vorige week niet gemaakt of niet af gekregen, dan kun je de [voorbeeld uitwerkingen](https://github.com/hogeschoolnovi/backend-springboot-vinylshop-repository-uitwerkingen) clonen.

# Opdrachtbeschrijving

Je gaat in deze opdracht verder werken aan de opdracht die je vorige week gemaakt hebt. 

Je gaat de entities uitbreiden met DTO's en mappers. 
Uiteraard ga je er ook zorgen dat jouw controllers en services met die DTO's kunnen werken.
        

# Randvoorwaarden

- Zorg dat je mappenstructuur aan de Maven voorwaarden voldoet
- Je mappen structuur heeft ten minste de packages: 
  - controllers
  - repositories
  - services
  - dtos
  - entities
- Je hebt twee controllers:
  - GenreController
  - PublisherController
- Je hebt twee service:
  - GenreService
  - PublisherService
- Je hebt twee repositories:
  - GenreRepository
  - PublisherRepository
- Je hebt drie entiteiten:
  - GenreEntity
  - PublisherEntity
  - BaseEntity
- Je hebt vier DTO's
  - GenreRequestDTO
  - GenreResponseDTO
  - PublisherRequestDTO
  - PublisherResponseDTO
- Je hebt twee mappers
  - GenreMapper
  - PublisherMapper
- Je hebt de juiste instellingen in je application.properties om de database connectie goed tot stand te brengen.
- Je hebt de juiste dependencies aan het project gekoppeld (ook voor validatie).
- Je hebt minimaal één record van beide entiteiten in je data.sql staan.
- Je hebt een export van Postman met up-to-date requests voor beide entiteiten
- Je hebt een eigen `RecordNotFoundException` en een `GlobalExceptionHandler` die deze en de DTO validatie exception opvangt en de gebruiker een goede berichtgeving terugstuurt.


# Stappenplan

## Stap 0 (packages)

Maak de juiste packages aan. De structuur van de nieuwe packages ziet er als volgt uit:

- dtos
  - genre
  - publisher
- mappers

## Stap 1 (Response DTO's)

Maak een `dto` map met daarin een een `genre` en een `publisher` map.

Maak een `GenreResponseDto` met de volgende velden:
- id
- name 
- description


Maak een `PublisherResponseDto` met de volgende velden:
- id
- name
- address
- contactDetails

## Stap 2 (Request DTO's)

Maak een `GenreRequestDto` met de volgende validatie regels: 
- De naam mag niet leeg zijn
- De naam mag niet langer dan 100 karakters zijn en niet korter dan 2
- De description mag niet langer dan 255 karakters zijn.

Maak een `PublisherRequestDto` met de volgende validatie regels:
- De naam mag niet leeg zijn
- De naam mag niet langer dan 50 karakters zijn.

Zorg er bij beide DTO's voor dat er ook een goede message gemaakt wordt.

## Stap 3 (POM)

Voeg de validatie dependency toe aan je pom.xml.

## Stap 4 (Mappers)


De mapper pakken we op een slimme manier aan door een algemene interface voor de mappers te maken.

De interface mag je als volgt definieren:

``` java
public interface DTOMapper<RESPONSE, REQUEST , T extends BaseEntity> {
    RESPONSE mapToDto(T model);

    List<RESPONSE> mapToDto(List<T> models);

    T mapToEntity(REQUEST genreModel);
}
```

Maak nu de `GenreDTOMapper` waarin je de volgende methodes vanuit de interface overschrijft:
- public GenreResponseDTO mapToDto(GenreEntity model)
- public List<GenreResponseDTO> mapToDto(List<GenreEntity> models)
- public GenreEntity mapToEntity(GenreRequestDTO genreModel)

Maak ook de `PublisherDTOMapper` waarin je de volgende methodes vanuit de interface overschrijft:
- public PublisherResponseDTO mapToDto(PublisherEntity publisher)
- public List<PublisherResponseDTO> mapToDto(List<PublisherEntity> publishers)
- public PublisherEntity mapToEntity(PublisherRequestDTO dto)

> Merk op dat elke DTOMapper-klasse dus twee methodes heeft met de naam "mapToDto". Één met een model als input en een dto als output en één met een List<model> als input en een List<dto> als output. Dit noemen we "method overloading".

Vergeet niet de juiste annotatie boven de Mapper klassen te zetten.

## Stap 5 (Service) 

Injecteer de `GenreDTOMapper` in de `GenreService` en injecteer de `PublisherDTOMapper` in de `PublisherService`.

Zorg dat alle methodes die nu een `GenreEntity` naar de controller returnen, aangepast worden zodat ze een `GenreResponseDTO` naar de controller returnen.

Maak hierbij gebruik van de `GenreDTOMapper`.

Hier is een voorbeeld van de aangepaste `createGenre` methode: 

```java
public GenreResponseDTO createGenre(GenreRequestDTO genreDTO) {
    GenreEntity genreEntity = genreDTOMapper.mapToEntity(genreDTO);
    genreEntity = genreRepository.save(genreEntity);
    return genreDTOMapper.mapToDto(genreEntity);
}

```

Pas de `PublisherService` op dezelfde manier aan.

## Stap 6 (Controller)

Zorg dat jouw controllers, en alle methodes (GET-, POST-, PUT- en DELETE-mappings) gebruik maken van de DTO's als input en als output.

Als voorbeeld hier de aangepaste POST mapping van de GenreController: 
```java
    @PostMapping
    public ResponseEntity<GenreResponseDTO> createGenre(@RequestBody @Valid GenreRequestDTO genreModel) {
        GenreResponseDTO newGenre = genreService.createGenre(genreModel);
        return ResponseEntity.created(urlHelper.getCurrentUrlWithId(newGenre.getId())).body(newGenre);
    }
```

Vergeet niet om de `@Valid` annotatie te gebruiken, anders worden je validatie regels niet gevalideerd.

## Stap 7 (Exceptions)

Maak een `exceptions` package.

Maak in de `exceptions` package een `RecordNotFoundException`.

Zorg er voor dat je op alle plekken waar je een `Optional<entiteit>` uit de repository ophaalt, je een `RecordNotFoundException` opgooit wanneer de Optional leeg blijkt te zijn. 

Maak in de `helpers` package een `GlobalExceptionHandler`-klasse met de juiste annotatie er boven. 

Maak in deze GlobalExceptionHandler-klasse ten minste 2 methodes. Één methode die de `MethodArgumentNotValidException` opvangt en één methode die de  `RecordNotFoundException` opvangt (wanneer de gebruiker een validatieregel overtreed door foute json input  mee te geven). De eerste methode mag als volgt implementeren: 

```java
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<List<String>> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex) {
        List<String> violations = ex
                .getBindingResult()
                .getFieldErrors()
                .stream()
                .map(FieldError::getDefaultMessage)
                .toList();
        return ResponseEntity.badRequest().body(violations);
    }
```	

De tweede methode mag je implementeren door een 404 statuscode te returnen. Een tekstuele berichtgeving daarbij is niet nodig, maar mag wel.

## Stap 8 (Postman)
Voeg een actuele export van je postman collectie toe aan de `resources` map.

Waarschijnlijk kun je hier dezelfde postman export voor gebruiken als in de vorige opdracht, maar controleer het wel even. 

Test vooral ook goed of je geen onverwachte `null` waardes terug krijgt, omdat je een mapper niet goed ingevuld hebt. 

Test ook wat er gebeurt als je een PostMan request verstuurd die de validatieregels overtreedt.

