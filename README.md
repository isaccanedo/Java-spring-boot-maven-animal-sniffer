## Animal Sniffer Maven Plugin

# 1. Introdução
Enquanto trabalhamos em Java, há momentos em que precisamos usar versões em vários idiomas ao mesmo tempo.

É comum precisar que nosso programa Java seja compatível em tempo de compilação com uma versão Java (digamos - Java 6), mas precisar usar uma versão diferente (digamos - Java 8) em nossas ferramentas de desenvolvimento e talvez uma versão diferente para executar o aplicativo .

Neste artigo rápido, vamos demonstrar como é fácil adicionar salvaguardas de incompatibilidade baseadas na versão Java e como o plugin Animal Sniffer pode ser usado para sinalizar esses problemas em tempo de compilação, verificando nosso projeto contra assinaturas geradas anteriormente.

# 2. Configurando -source e -target do compilador Java
Vamos começar com um projeto Maven hello world - em que estamos usando Java 7 em nossa máquina local, mas gostaríamos de implantar o projeto no ambiente de produção que ainda usa Java 6.

Nesse caso, podemos configurar o plug-in do compilador Maven com os campos de origem e destino apontando para Java 6.

O campo “source” é usado para especificar a compatibilidade com as alterações da linguagem Java e o campo “target” é usado para especificar a compatibilidade com as alterações JVM.

Vejamos agora a configuração do compilador Maven de pom.xml:

```
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.7.0</version>
	<configuration>
            <source>1.6</source>
            <target>1.6</target>
	</configuration>
    </plugin>
</plugins>
```

Com o Java 7 em nossa máquina local e o código Java imprimindo “hello world” no console, se prosseguirmos e construirmos este projeto usando Maven, ele construirá e funcionará corretamente em uma caixa de produção executando Java 6.

# 3. Apresentando incompatibilidades de API
Vejamos agora como é fácil introduzir acidentalmente a incompatibilidade de API.

Digamos que começamos a trabalhar em algum novo requisito e usamos alguns recursos da API do Java 7 que não estavam presentes no Java 6.

Vejamos o código-fonte atualizado:
```
public static void main(String[] args) {
    System.out.println("Hello World!");
    System.out.println(StandardCharsets.UTF_8.name());
}
```

java.nio.charset.StandardCharsets foi introduzido no Java 7.

Se prosseguirmos e executarmos o build do Maven, ele ainda será compilado com êxito, mas falhará no tempo de execução com erro de ligação em uma caixa de produção com Java 6 instalado.

A documentação do Maven menciona essa armadilha e recomenda o uso do plugin Animal Sniffer como uma das opções.

# 4. Compatibilidades da API de relatórios
O plugin Animal Sniffer oferece dois recursos principais:

Gerando assinaturas do Java runtime
1. Verificando um projeto em relação às assinaturas de API
2. Vamos agora modificar o pom.xml para incluir o plug-in:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>animal-sniffer-maven-plugin</artifactId>
    <version>1.16</version>
    <configuration>
        <signature>
            <groupId>org.codehaus.mojo.signature</groupId>
            <artifactId>java16</artifactId>
            <version>1.0</version>
        </signature>
    </configuration>
    <executions>
        <execution>
            <id>animal-sniffer</id>
            <phase>verify</phase>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Aqui, a seção de configuração do Animal Sniffer se refere a uma assinatura de tempo de execução Java 6 existente. Além disso, a seção de execução verifica e verifica o código-fonte do projeto em relação à assinatura fornecida e sinaliza se algum problema for encontrado.

Se prosseguirmos e construirmos o projeto Maven, a construção falhará com o erro de verificação de assinatura de relatório do plug-in, conforme esperado:

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.codehaus.mojo:animal-sniffer-maven-plugin:1.16:check 
(animal-sniffer) on project example-animal-sniffer-mvn-plugin: Signature errors found.
Verify them and ignore them with the proper annotation if needed.
```

# 5. Conclusão
Neste tutorial, exploramos o plug-in Maven Animal Sniffer e como ele pode ser usado para relatar incompatibilidades relacionadas à API, se houver, no momento da construção.
