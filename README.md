# Exercícios Java/Javalin
## Emanuel Alves de Cristo
---

## Parte 1: Executando `RandomAdviceService` no meu computador

![ex1](https://github.com/elc117/apresentacao-bim2-2026a-EmanuelCristo/blob/main/Exemplo_Advice.gif)

## Parte 2: Analisando o código
### Código 1 `randomAdviceService`

```java
package demo;

import io.javalin.Javalin;
import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

public class RandomAdviceService {
    private static final List<String> ADVICES = List.of(
            "It always seems impossible, until it's done.",
            "To cleanly remove the seed from an Avocado, lay a knife firmly across it, and twist.",
            "Fail. Fail again. Fail better.",
            "Play is the true mother of invention.",
            "Remedy tickly coughs with a drink of honey, lemon and water as hot as you can take.");

    public static void main(String[] a) {
        int p = Integer.parseInt(System.getenv().getOrDefault("PORT", "3000"));
        var app = Javalin.create();
        app.get("/advice", ctx -> {
            var i = ThreadLocalRandom.current().nextInt(ADVICES.size());
            ctx.result(ADVICES.get(i));
        });
        app.start(p);
    }
}
```
### O que eu já conhecia
* Sintaxe básica do Java: `public class RandomAdviceService`, `private static final List<String>`, `public static void main(String[] a)`, `int`, `var`.
* Chamada do Método `Integer.parseInt`
### O que era desconhecido
* Coleções Imutáveis: `List.of()`
  * Cria uma lista imutável contendo os elementos especificados. Possuí tamanho fixo e não permite tamanhos nulos.
---
* Expressões Lambda em Java: `app.get("/advice", ctx -> { ... });`
* "Sintaxe do Famework": `(System.getenv().getOrDefault("PORT", "3000"));`, `Javalin.create();`,
  * `System.getenv()` tenta ler a variável de ambiente PORT. Se não achar, usa a porta 3000 por padrão.
---
* Classe `ThreadLocalRandom`: `ThreadLocalRandom.current().nextInt(ADVICES.size())`
  * Um gerador de números aleatórios isolado para a thread atual.

### Código 2 `TeamRecommendationService` (gerado segunda-feira)

```java
package demo;

import io.javalin.Javalin;
import io.javalin.http.HttpStatus;

import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class TeamRecommendationService {

    public static class Person {
        public Integer id; // nullable on create
        public String name;
        public String attributes; // JSON string of attributes

        public Person() {}
        public Person(Integer id, String name, String attributes) {
            this.id = id;
            this.name = name;
            this.attributes = attributes;
        }
    }

    private static Connection conn;

    public static void main(String[] args) throws Exception {
        int port = Integer.parseInt(System.getenv().getOrDefault("PORT", "3000"));
        Javalin app = Javalin.create().start(port);

        // Conectar ao banco de dados SQLite
        conn = DriverManager.getConnection("jdbc:sqlite:team_recommendation.db");
        try (Statement st = conn.createStatement()) {
            st.execute("""
                CREATE TABLE IF NOT EXISTS people (
                    id INTEGER PRIMARY KEY AUTOINCREMENT,
                    name TEXT NOT NULL,
                    attributes TEXT NOT NULL
                )
            """);
        }

        // Endpoint para listar todas as pessoas
        app.get("/people", ctx -> {
            List<Person> people = new ArrayList<>();
            try (Statement st = conn.createStatement();
                 ResultSet rs = st.executeQuery("SELECT * FROM people")) {
                while (rs.next()) {
                    people.add(new Person(
                        rs.getInt("id"),
                        rs.getString("name"),
                        rs.getString("attributes")
                    ));
                }
            }
            ctx.json(people);
        });

        // Endpoint para adicionar uma pessoa
        app.post("/people", ctx -> {
            Person person = ctx.bodyAsClass(Person.class);
            try (PreparedStatement pst = conn.prepareStatement(
                "INSERT INTO people (name, attributes) VALUES (?, ?)",
                Statement.RETURN_GENERATED_KEYS
            )) {
                pst.setString(1, person.name);
                pst.setString(2, person.attributes);
                pst.executeUpdate();
                try (ResultSet rs = pst.getGeneratedKeys()) {
                    if (rs.next()) {
                        person.id = rs.getInt(1);
                    }
                }
            }
            ctx.status(HttpStatus.CREATED).json(person);
        });

        // Endpoint para atualizar uma pessoa
        app.put("/people/:id", ctx -> {
            int id = Integer.parseInt(ctx.pathParam("id"));
            Person person = ctx.bodyAsClass(Person.class);
            try (PreparedStatement pst = conn.prepareStatement(
                "UPDATE people SET name = ?, attributes = ? WHERE id = ?"
            )) {
                pst.setString(1, person.name);
                pst.setString(2, person.attributes);
                pst.setInt(3, id);
                if (pst.executeUpdate() == 0) {
                    ctx.status(HttpStatus.NOT_FOUND);
                } else {
                    ctx.json(person);
                }
            }
        });

        // Endpoint para remover uma pessoa
        app.delete("/people/:id", ctx -> {
            int id = Integer.parseInt(ctx.pathParam("id"));
            try (PreparedStatement pst = conn.prepareStatement(
                "DELETE FROM people WHERE id = ?"
            )) {
                pst.setInt(1, id);
                if (pst.executeUpdate() == 0) {
                    ctx.status(HttpStatus.NOT_FOUND);
                } else {
                    ctx.status(HttpStatus.NO_CONTENT);
                }
            }
        });
    }
}
```
## Executando `TeamRecommendationService`

![ex2](https://github.com/elc117/apresentacao-bim2-2026a-EmanuelCristo/blob/main/exemplo_segunda.gif)

### O que eu já conhecia
* Classes Aninhadas: `public static class Person`é definida dentro de `public class TeamRecommendationService`.
* Sobrecarga de Construtores: `public Person()` `public Person(Integer id, String name, String attributes)` permitindo instanciar o objeto com ou sem dados iniciais.
* Coleções Mutáveis: `List<Person> people = new ArrayList<>();` cria lista dinâmicas (contrário do `List.of()`).
* Estruturas de Repetição e Condicionais: `while` `if/else`
* Padrão RESTful: `(GET, POST, PUT, DELETE)`

Referências
* https://medium.com/@mgm06bm/list-of-vs-arrays-aslist-7e2f7af64361
* https://medium.com/@luizhenrique.se/conhecendo-o-javalin-um-framework-para-java-e-kotlin-parte-1-f2ad42130e0
* https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ThreadLocalRandom.html
* https://www.devmedia.com.br/como-usar-funcoes-lambda-em-java/32826
* https://javalin.io/blog/static-methods-within-lambdas

