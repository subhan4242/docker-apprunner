# Oppgave: Fra Spring Boot til AWS App Runner 

I denne oppgaven skal du:

1. Lage en ny **Spring Boot-applikasjon** via [Spring Initializr](https://start.spring.io/)
2. Containerisere applikasjonen med Docker
3. Publisere imaget til **Amazon ECR**
4. Starte en **AWS App Runner-tjeneste** fra imaget ditt

---

## Steg 1: Opprett Spring Boot-applikasjonen

- Gå til [https://start.spring.io/](https://start.spring.io/)
- Velg:
    - **Project:** Maven
    - **Language:** Java
    - **Spring Boot:** 3.x (seneste versjon)
    - **Packaging:** Jar
    - **Java:** 21
    - Legg til dependency: `Spring Web` (for REST API)
- Trykk **Generate**, last ned zip, og pakk ut prosjektet inn i repoet ditt (i Codespaces).

Lag en enkel controller, f.eks. `HelloController.java`:

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot on App Runner!";
    }
}
```

## Steg 1: Opprett Spring Boot-applikasjonen

- Gå til [https://start.spring.io/](https://start.spring.io/)
- Velg:
  - **Project:** Maven
  - **Language:** Java
  - **Spring Boot:** 3.x (seneste versjon)
  - **Packaging:** Jar
  - **Java:** 21
  - Legg til dependency: `Spring Web` (for REST API)
- Trykk **Generate**, last ned zip, og pakk ut prosjektet inn i repoet ditt (i Codespaces).  

Lag en enkel controller, f.eks. `HelloController.java`:

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @GetMapping("/")
    public String hello() {
        return "Hello from Spring Boot on App Runner!";
    }
}
```

Bygg og test applikasjonen i Codespaces:

```shell
./mvnw spring-boot:run
```

Sjekk 

```shell
curl http://localhost:8080/
```

## Steg 2: Lag en Dockerfile

Opprett en **Dockerfile** i rotmappen:

```dockerfile
FROM eclipse-temurin:21-jdk AS build
WORKDIR /app
COPY . .
RUN ./mvnw -q package -DskipTests

FROM eclipse-temurin:21-jre
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Bygg lokalt i Codespaces og kjør en test:

```bash
docker build -t springboot-demo .
docker run -p 8080:8080 springboot-demo
```

## Steg 3: Push til Amazon ECR

1. Lag ECR-repoet ditt:

   ```bash
   aws ecr create-repository --repository-name springboot-demo
   ```

2. Logg inn i ECR:

   ```bash
   aws ecr get-login-password --region eu-west-1 \
     | docker login --username AWS --password-stdin <account-id>.dkr.ecr.eu-west-1.amazonaws.com
   ```

3. Tagg og push:

   ```bash
   docker tag springboot-demo:latest <account-id>.dkr.ecr.eu-west-1.amazonaws.com/springboot-demo:latest
   docker push <account-id>.dkr.ecr.eu-west-1.amazonaws.com/springboot-demo:latest
   ```


## Steg 4: Lag en App Runner-tjeneste

1. Gå til **AWS Console → App Runner → Create service**
2. Velg **Container registry → Amazon ECR**
3. Finn repoet `springboot-demo` og velg imaget ditt
4. Sett port til `8080`
5. Deploy 🎉

Når tjenesten kjører får du en **public URL** som du kan åpne i nettleseren.

---

## Oppsummering

Du har nå:

* Laget en Spring Boot-app med **Spring Initializr**
* Bygget og containerisert applikasjonen med **Docker**
* Publisert imaget til **Amazon ECR**
* Deployet applikasjonen som en **App Runner-tjeneste**
```
