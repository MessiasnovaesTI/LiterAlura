# LiterAlura
Praticando Spring Boot: Challenge LiterAlura
### Desenvolvimento do Projeto LiterAlura

### 1. Configuração do Projeto Spring

#### a. Configuração do Spring Initializr
- **Project**: Maven Project
- **Language**: Java
- **Spring Boot Version**: 2.5.4
- **Project Metadata**:
  - Group: `com.alura`
  - Artifact: `literalura`
  - Name: `literalura`
  - Package name: `com.alura.literalura`
  - Packaging: `jar`
  - Java Version: `11`

#### b. Dependências
- **Spring Web**: para criar os endpoints REST
- **Spring Data JPA**: para persistência de dados
- **PostgreSQL Driver**: para conexão com o banco de dados PostgreSQL

### 2. Configuração do Banco de Dados
No arquivo `application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/literalura
spring.datasource.username=postgres
spring.datasource.password=your_password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
```

### 3. Estrutura do Projeto

#### a. Modelos
- **Livro**
- **Autor**

```java
@Entity
public class Livro {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String titulo;
    private String autor;
    private String idioma;
    private int downloads;

    // Getters and Setters
}
```

```java
@Entity
public class Autor {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private LocalDate dataNascimento;
    private LocalDate dataFalecimento;

    // Getters and Setters
}
```

#### b. Repositórios
```java
public interface LivroRepository extends JpaRepository<Livro, Long> {
    List<Livro> findByIdioma(String idioma);
    List<Livro> findByTituloContaining(String titulo);
}

public interface AutorRepository extends JpaRepository<Autor, Long> {
    List<Autor> findByNomeContaining(String nome);
    List<Autor> findByDataNascimentoBeforeAndDataFalecimentoAfter(LocalDate data);
}
```

#### c. Serviços
```java
@Service
public class LivroService {
    @Autowired
    private LivroRepository livroRepository;

    public Livro salvar(Livro livro) {
        return livroRepository.save(livro);
    }

    public List<Livro> listarTodos() {
        return livroRepository.findAll();
    }

    public List<Livro> buscarPorIdioma(String idioma) {
        return livroRepository.findByIdioma(idioma);
    }
    
    public List<Livro> buscarPorTitulo(String titulo) {
        return livroRepository.findByTituloContaining(titulo);
    }
}

@Service
public class AutorService {
    @Autowired
    private AutorRepository autorRepository;

    public Autor salvar(Autor autor) {
        return autorRepository.save(autor);
    }

    public List<Autor> listarTodos() {
        return autorRepository.findAll();
    }

    public List<Autor> buscarPorNome(String nome) {
        return autorRepository.findByNomeContaining(nome);
    }

    public List<Autor> buscarPorPeriodo(LocalDate data) {
        return autorRepository.findByDataNascimentoBeforeAndDataFalecimentoAfter(data);
    }
}
```

#### d. Controladores
```java
@RestController
@RequestMapping("/api/livros")
public class LivroController {
    @Autowired
    private LivroService livroService;

    @GetMapping
    public List<Livro> listarTodos() {
        return livroService.listarTodos();
    }

    @GetMapping("/{id}")
    public Livro buscarPorId(@PathVariable Long id) {
        return livroService.buscarPorId(id);
    }

    @GetMapping("/buscar")
    public List<Livro> buscarPorTitulo(@RequestParam String titulo) {
        return livroService.buscarPorTitulo(titulo);
    }

    @PostMapping
    public Livro salvar(@RequestBody Livro livro) {
        return livroService.salvar(livro);
    }

    @GetMapping("/idioma/{idioma}")
    public List<Livro> buscarPorIdioma(@PathVariable String idioma) {
        return livroService.buscarPorIdioma(idioma);
    }
}

@RestController
@RequestMapping("/api/autores")
public class AutorController {
    @Autowired
    private AutorService autorService;

    @GetMapping
    public List<Autor> listarTodos() {
        return autorService.listarTodos();
    }

    @GetMapping("/{id}")
    public Autor buscarPorId(@PathVariable Long id) {
        return autorService.buscarPorId(id);
    }

    @GetMapping("/buscar")
    public List<Autor> buscarPorNome(@RequestParam String nome) {
        return autorService.buscarPorNome(nome);
    }

    @GetMapping("/periodo/{data}")
    public List<Autor> buscarPorPeriodo(@PathVariable String data) {
        LocalDate localDate = LocalDate.parse(data);
        return autorService.buscarPorPeriodo(localDate);
    }
}
```

### 4. Consumo da API Gutendex

#### a. Dependência no `pom.xml`
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

#### b. Serviço para consumo da API
```java
@Service
public class GutendexService {
    private final WebClient webClient;

    public GutendexService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://gutendex.com/books").build();
    }

    public Mono<Livro> buscarLivroPorTitulo(String titulo) {
        return webClient.get()
            .uri(uriBuilder -> uriBuilder.queryParam("title", titulo).build())
            .retrieve()
            .bodyToMono(Livro.class);
    }
}
```

#### c. Endpoint de busca de livro pelo título
```java
@GetMapping("/buscarApi")
public Livro buscarLivroNaApi(@RequestParam String titulo) {
    return gutendexService.buscarLivroPorTitulo(titulo).block();
}
```

### 5. Testando a Aplicação
Use a interface do terminal para interagir com o sistema, conforme os casos descritos.
