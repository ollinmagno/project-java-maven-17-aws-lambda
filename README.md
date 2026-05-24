# ☕ Hello Java 17 — AWS Lambda com Maven

Projeto de exemplo demonstrando como empacotar e executar uma função **AWS Lambda** usando **Java 17** e **Maven**, com o plugin `maven-shade` para geração do JAR de deploy.

---

## 📋 Sumário

- [Visão Geral](#visão-geral)
- [Pré-requisitos](#pré-requisitos)
- [Código da Função](#código-da-função)
- [Build com Maven](#build-com-maven)
- [Deploy na AWS Lambda](#deploy-na-aws-lambda)
- [Testando a Função](#testando-a-função)
- [Resultado Esperado](#resultado-esperado)
- [Referências](#referências)

---

## Visão Geral

Esta função Lambda recebe uma `String` como input, loga a versão do JDK em uso e retorna o texto convertido para **maiúsculas**.

| Campo              | Valor                          |
|--------------------|-------------------------------|
| Runtime            | Java 17                        |
| Handler            | `dev.ollin.hj17.SimpleHandler` |
| Input type         | `String`                       |
| Output type        | `String`                       |
| Build tool         | Maven + maven-shade-plugin     |

---

## Pré-requisitos

- [Java 17 (JDK)](https://adoptium.net/)
- [Apache Maven 3.6+](https://maven.apache.org/)
- Conta AWS com permissões de Lambda

## Código da Função

**`src/main/java/dev/ollin/hj17/SimpleHandler.java`**

```java
package dev.ollin.hj17;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.LambdaLogger;
import com.amazonaws.services.lambda.runtime.RequestHandler;

public class SimpleHandler implements RequestHandler<String, String> {

    public String handleRequest(String input, Context context) {
        LambdaLogger logger = context.getLogger();
        logger.log("JDK Version: " + System.getProperty("java.version"));
        return input.toUpperCase();
    }
}
```

A função implementa `RequestHandler<String, String>` do SDK da AWS Lambda e:

Obtém o logger do contexto da Lambda e loga a versão do JDK em execução via `System.getProperty("java.version")` e retorna o input convertido.

---

## Build com Maven

### Configuração do `pom.xml`

O plugin `maven-shade` empacota todas as dependências em um único JAR ("fat JAR" / "uber JAR"), necessário para o deploy na AWS Lambda:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <configuration>
                <createDependencyReducedPom>false</createDependencyReducedPom>
            </configuration>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

> **Por que `maven-shade`?** A AWS Lambda exige um JAR único com todas as dependências incluídas. O shade-plugin faz esse empacotamento automaticamente na fase `package`.

### Gerando o JAR

```bash
mvn clean package
```

O JAR gerado estará em:

```
target/helloJava17-1.0-SNAPSHOT.jar
```

---

## Deploy na AWS Lambda

### Via Console AWS

1. Acesse **AWS Lambda → Criar função**
2. Selecione **"Criar do zero"**
3. Configure:
   - **Nome da função:** `helloJava17`
   - **Runtime:** `Java 17`
   - **Arquitetura:** `x86_64`
4. Em **"Código"**, faça upload do JAR gerado em `target/`
5. Em **"Configuração de runtime"**, defina o handler:

```
dev.ollin.hj17.SimpleHandler::handleRequest
```

## Testando a Função

### Via Console AWS

No painel da função, vá em **Test** e use o seguinte evento de teste:

```json
"hello, java 17"
```

Clique em **Test** para executar.

## Resultado Esperado

Após a execução bem-sucedida, você verá:

**Saída da função:**
```
"HELLO, JAVA 17"
```

**Log output (CloudWatch):**
```
START RequestId: cbb2607b-87c9-4030-b437-c85ba6363dc1 Version: $LATEST
JDK Version: 17.0.18
END RequestId: cbb2607b-87c9-4030-b437-c85ba6363dc1
REPORT RequestId: cbb2607b-87c9-4030-b437-c85ba6363dc1
  Duration: 30.06 ms
  Billed Duration: 403 ms
  Memory Size: 512 MB
  Max Memory Used: 95 MB
  Init Duration: 372.88 ms
```

### Métricas de Execução

| Métrica              | Valor      | Observação                              |
|----------------------|------------|-----------------------------------------|
| Duration             | 30.06 ms   | Tempo real de execução                  |
| Billed Duration      | 403 ms     | Inclui o cold start                     |
| Init Duration        | 372.88 ms  | Cold start — JVM initialization         |
| Max Memory Used      | 95 MB      | Pico de uso de memória                  |
| Resources Configured | 512 MB     | Memória alocada para a função           |

> **Nota sobre Cold Start:** O `Init Duration` de ~373ms é característico do Java na Lambda (inicialização da JVM). Em invocações subsequentes ("warm"), apenas o `Duration` de ~30ms é cobrado.

---

## Referências

- [AWS Lambda com Java — Documentação oficial](https://docs.aws.amazon.com/lambda/latest/dg/lambda-java.html)
- [AWS Lambda Java SDK](https://github.com/aws/aws-lambda-java-libs)
- [Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/)
- [Amazon Corretto 17 (JDK)](https://aws.amazon.com/corretto/)

---

*Desenvolvido com ☕ Java 17 + AWS Lambda*
