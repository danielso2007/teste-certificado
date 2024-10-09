# Teste Certificado

Apenas para teste de cerificado no spring-boot.

## Execução local

O que eu testei localmente.

```shell
openssl genpkey -algorithm RSA -out teste-local.key

openssl req -new -x509 -key teste-local.key -out certificado.crt -days 365

openssl pkcs12 -export -in certificado.crt -inkey teste-local.key -out certificado.p12 -name "local-certificado"

keytool -importkeystore \
    -deststorepass P0o9i8u7 \
    -destkeypass qawsedrf \
    -destkeystore keystore.jks \
    -srckeystore certificado.p12 \
    -srcstoretype PKCS12 \
    -alias "local-certificado"

keytool -list -v -keystore certificado.p12 -storepass qawsedrf -storetype PKCS12
```

Ao executar a aplicação, chamar [https://localhost:8081/](https://localhost:8081/)

# Passos para criar

Para criar certificados SSL usando o Let's Encrypt, você pode usar a ferramenta **Certbot**. Aqui está um passo a passo básico:

## 1\. Instalar o Certbot

A maneira de instalar o Certbot varia conforme o sistema operacional. Para **Ubuntu**, por exemplo:

```shell
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

Esse comando instala o Certbot junto com o plugin para o Nginx. Se você usa Apache, troque nginx por apache.

## 2\. Obter o certificado SSL

Após a instalação, você pode usar o Certbot para gerar o certificado. Para Nginx, o comando é:

```shell
sudo certbot --nginx -d seu-dominio.com -d www.seu-dominio.com
```

Para Apache:

```shell
sudo certbot --apache -d seu-dominio.com -d www.seu-dominio.com
```

## 3\. Responder às perguntas do Certbot

O Certbot fará algumas perguntas, como:

- Se deseja redirecionar HTTP para HTTPS (recomendado).

## 4\. Testar a renovação automática

O Certbot configura automaticamente a renovação de certificados, mas é bom testar:

```shell
sudo certbot renew --dry-run
```

## 5\. Configurar a renovação automática

A maioria das distribuições já configura um cron job ou timer do systemd para renovar automaticamente os certificados antes de expirarem. Contudo, é possível adicionar manualmente ao crontab:

```shell
0 0,12 \* \* \* root certbot renew --quiet
```

## Como usar esse certificado na JVM do Java?

Para usar o certificado SSL emitido pelo Let's Encrypt em uma aplicação Java, você precisa adicioná-lo ao **Java KeyStore (JKS)**, que é o formato de armazenamento de chaves e certificados utilizado pela JVM. Aqui estão os passos:

## 1\. Converter o Certificado para o Formato PFX/PKCS12

O Let's Encrypt gera o certificado no formato PEM, mas o KeyStore da JVM trabalha com o formato PFX/PKCS12. Você pode converter o certificado usando o **OpenSSL**.

```shell
openssl pkcs12 -export \
    -in /etc/letsencrypt/live/seu-dominio.com/fullchain.pem \
    -inkey /etc/letsencrypt/live/seu-dominio.com/privkey.pem \
    -out certificado.pfx \
    -name "seu-alias"
```

- `fullchain.pem`: Contém o certificado da sua aplicação e o certificado da cadeia intermediária.
- `privkey.pem`: Chave privada do certificado.
- `certificado.pfx`: Nome do arquivo de saída no formato PKCS12.
- `seu-alias`: Nome que você quer dar ao certificado no KeyStore.

## 2\. Importar o Certificado no Java KeyStore (JKS)

Após converter o certificado, use a ferramenta keytool para importar o arquivo PKCS12 gerado no KeyStore.

```shell
keytool -importkeystore \
    -deststorepass senha-do-keystore \
    -destkeypass senha-da-chave \
    -destkeystore keystore.jks \
    -srckeystore certificado.pfx \
    -srcstoretype PKCS12 \
    -alias "seu-alias"
```

- `deststorepass`: Senha do KeyStore.
- `destkeypass`: Senha da chave.
- `keystore.jks`: Nome do arquivo de saída do KeyStore.
- `certificado.pfx`: Arquivo PFX gerado no passo anterior.
- `alias`: Alias do certificado.

## 3\. Configurar o Tomcat ou Outro Servidor com o KeyStore

Se você estiver usando um servidor como o Tomcat, adicione a configuração do KeyStore no arquivo server.xml.

```xml
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
    maxThreads="150" scheme="https" secure="true"
    keystoreFile="/caminho/para/keystore.jks"
    keystorePass="senha-do-keystore"
    clientAuth="false" sslProtocol="TLS"/>
```

## 4\. Verificar o Certificado

Após configurar o servidor, reinicie-o e verifique se o certificado está funcionando corretamente acessando o domínio com HTTPS.

## Como adicionar certificados em uma aplicação spring boot?

Para adicionar certificados SSL em uma aplicação **Spring Boot**, o processo envolve configurar o **Tomcat embutido** (ou outro servidor) para usar o certificado. Aqui estão os passos detalhados:

## 1\. Converter o Certificado para o Formato PKCS12

Primeiro, converta o certificado e a chave privada gerados pelo Let's Encrypt para o formato PKCS12, que o Spring Boot usa por padrão.

```shell
openssl pkcs12 -export \
    -in /etc/letsencrypt/live/seu-dominio.com/fullchain.pem \
    -inkey /etc/letsencrypt/live/seu-dominio.com/privkey.pem \
    -out certificado.p12 \
    -name "seu-alias" \
    -passout pass:senha-do-certificado
```

- `fullchain.pem`: Certificado e cadeia intermediária.
- `privkey.pem`: Chave privada.
- `certificado.p12`: Nome do arquivo de saída PKCS12.
- `seu-alias`: Nome do alias no keystore.
- `senha-do-certificado`: Senha para proteger o arquivo PKCS12.

## 2\. Configurar o Application Properties/Configuration

Agora, configure o Spring Boot para usar o certificado SSL. Adicione as seguintes propriedades ao arquivo `application.properties` ou `application.yml`:

**Usando application.properties:**

```properties
server.port=8443
server.ssl.key-store=classpath:certificado.p12
server.ssl.key-store-password=senha-do-certificado
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=seu-alias
```

**Usando application.yml:**

```yml
server:
  port: 8443
  ssl:
    key-store: classpath:certificado.p12
    key-store-password: senha-do-certificado
    key-store-type: PKCS12
    key-alias: seu-alias
```

## 3\. Colocar o Certificado no Classpath

O arquivo certificado.p12 gerado deve ser colocado na pasta **resources** do seu projeto Spring Boot, para que ele possa ser carregado no **classpath**.

## 4\. Executar a Aplicação com SSL

Com as configurações prontas, quando você iniciar a aplicação Spring Boot, ela irá ouvir na porta **8443** usando o certificado SSL configurado. Acesse a aplicação via `https://seu-dominio.com:8443`.

## 5\. Testar a Aplicação

Depois de rodar a aplicação, teste o acesso HTTPS no navegador ou com ferramentas como curl:

```shell
curl -v https://seu-dominio.com:8443
```

Isso confirmará que o certificado está ativo e funcionando corretamente.

## Como criar um certificado Let's Encrypt para adicionar na JVM?

Para criar um certificado Let's Encrypt e usá-lo na JVM, você pode seguir os seguintes passos, que envolvem gerar o certificado com o Let's Encrypt e convertê-lo para o formato compatível com o **Java KeyStore (JKS)**.

## Passo a Passo

### **1\. Instalar o Certbot**

Primeiro, instale o **Certbot** no seu servidor para gerar os certificados Let's Encrypt. Em um sistema baseado em **Ubuntu**:

```shell
sudo apt update
sudo apt install certbot
```

Se estiver usando **Nginx** ou **Apache**, você também pode instalar os plugins para integração automática, como:

```shell
sudo apt install python3-certbot-nginx
```

### **2\. Gerar o Certificado com Let's Encrypt**

Use o Certbot para gerar o certificado SSL para o seu domínio. Se você estiver usando Nginx ou Apache, o Certbot pode configurar o servidor automaticamente:

- Para **Nginx**:

```shell
sudo certbot --nginx -d seu-dominio.com -d www.seu-dominio.com
```

- Para **Apache**:

```shell
sudo certbot --apache -d seu-dominio.com -d www.seu-dominio.com
```

Caso não esteja usando Nginx ou Apache, você pode gerar o certificado de forma manual com o comando abaixo. Isso criará o certificado e a chave privada em um diretório especificado:

```shell
sudo certbot certonly --standalone -d seu-dominio.com
```

O certificado será gerado em `/etc/letsencrypt/live/seu-dominio.com/`.



### **3\. Converter o Certificado para PKCS12**

O **Java KeyStore (JKS)** não aceita os arquivos gerados diretamente pelo Certbot. Você precisa converter os arquivos .pem para o formato **PKCS12**, que é suportado pela JVM.

Use o **OpenSSL** para fazer isso:

```shell
openssl pkcs12 -export \
    -in /etc/letsencrypt/live/seu-dominio.com/fullchain.pem \
    -inkey /etc/letsencrypt/live/seu-dominio.com/privkey.pem \
    -out certificado.p12 \
    -name "seu-alias" \
    -CAfile /etc/letsencrypt/live/seu-dominio.com/chain.pem \
    -caname "CA" \
    -passout pass:senha-do-certificado
```

Esse comando faz o seguinte:

- `fullchain.pem`: Contém o certificado da aplicação e o intermediário.
- `privkey.pem`: Contém a chave privada do certificado.
- `certificado.p12`: Nome do arquivo PKCS12 gerado.
- `seu-alias`: Nome que você quer atribuir ao certificado.
- `senha-do-certificado`: Senha para proteger o arquivo PKCS12.

### **4\. Importar o Certificado no Java KeyStore (JKS)**

Agora, use a ferramenta **keytool** para importar o arquivo PKCS12 no formato **JKS**, que pode ser lido pela JVM:

```shell
keytool -importkeystore \
    -deststorepass senha-do-keystore \
    -destkeypass senha-da-chave \
    -destkeystore keystore.jks \
    -srckeystore certificado.p12 \
    -srcstoretype PKCS12 \
    -alias "seu-alias"
```

Aqui:

- `keystore.jks`: Nome do arquivo JKS a ser criado.
- `senha-do-keystore`: Senha do KeyStore.
- `senha-da-chave`: Senha da chave privada no KeyStore.
- `certificado.p12`: Arquivo PKCS12 que foi gerado.
- `alias`: Alias para o certificado.

### **5\. Configurar o Servidor JVM (Ex: Tomcat)**

Se você estiver usando o Tomcat, adicione a configuração do KeyStore no arquivo server.xml:

```xml
<Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
    maxThreads="150" scheme="https" secure="true"
    keystoreFile="/caminho/para/keystore.jks"
    keystorePass="senha-do-keystore"
    clientAuth="false" sslProtocol="TLS"/>
```

## 6\. Configurar a Renovação Automática

Para garantir que o certificado seja renovado automaticamente, adicione uma tarefa cron para renovar o certificado e converter o PKCS12:

```shell
0 3 * * * certbot renew --post-hook "openssl pkcs12 -export -in /etc/letsencrypt/live/seu-dominio.com/fullchain.pem -inkey /etc/letsencrypt/live/seu-dominio.com/privkey.pem -out /caminho/para/certificado.p12 -name seu-alias -passout pass:senha-do-certificado && keytool -importkeystore -deststorepass senha-do-keystore -destkeypass senha-da-chave -destkeystore /caminho/para/keystore.jks -srckeystore /caminho/para/certificado.p12 -srcstoretype PKCS12 -alias seu-alias"
```

Esse cron job será executado a cada 3 meses para renovar o certificado e recriar os arquivos no formato PKCS12 e JKS.

## Como verificar se o certificado está dentro de uma JVM?

Para verificar se o certificado foi corretamente instalado e está sendo utilizado corretamente dentro de uma JVM, você pode seguir alguns métodos de verificação. Aqui estão as abordagens mais comuns:

## 1\. Usando keytool para Verificar o Certificado no KeyStore

O **keytool** pode ser usado para inspecionar o conteúdo do **Java KeyStore (JKS)** ou de um arquivo **PKCS12** para garantir que o certificado foi importado corretamente.

### **Exemplo de Verificação do KeyStore**

Se o arquivo **KeyStore** está no formato **JKS**, você pode listar os certificados armazenados com o seguinte comando:

```shell
keytool -list -v -keystore caminho/para/keystore.jks -storepass senha-do-keystore
```

Se o arquivo estiver no formato **PKCS12**:

```shell
keytool -list -v -keystore caminho/para/certificado.p12 -storepass senha-do-certificado -storetype PKCS12
```

Este comando exibe detalhes sobre o certificado, como:

- **Alias**: Nome que identifica o certificado.
- **Tipo do certificado**: RSA, DSA, etc.
- **Validade**: Período de validade do certificado.
- **Emissor**: Informações da autoridade certificadora (CA).
- **Proprietário**: Para quem o certificado foi emitido (o **CN**, ou nome comum).
- **Cadeia de Certificação**: Se o certificado é parte de uma cadeia.

Verifique se as informações retornadas estão corretas e se o certificado está presente com a validade e os detalhes esperados.

## 2\. Verificar o Certificado em Tempo de Execução

Você também pode verificar programaticamente se um **KeyStore** foi carregado corretamente na JVM, utilizando o **SSLContext** e o **TrustManager**. Aqui está um exemplo de como fazer isso em código Java:

### **Exemplo em Código Java**

```java
import javax.net.ssl.KeyManagerFactory;
import javax.net.ssl.SSLContext;
import java.io.FileInputStream;
import java.security.KeyStore;

public class SSLVerification {
    public static void main(String\[\] args) throws Exception {
        // Carregar o KeyStore
        FileInputStream fis = new FileInputStream("caminho/para/keystore.jks");
        KeyStore keyStore = KeyStore.getInstance("JKS");
        keyStore.load(fis, "senha-do-keystore".toCharArray());

        // Verificar se o certificado está presente
        if (keyStore.isKeyEntry("alias-certificado")) {
            System.out.println("O certificado foi encontrado no KeyStore.");
        } else {
            System.out.println("O certificado não foi encontrado no KeyStore.");
        }

        // Criar um SSLContext com o KeyStore
        KeyManagerFactory keyManagerFactory = KeyManagerFactory.getInstance("SunX509");
        keyManagerFactory.init(keyStore, "senha-da-chave".toCharArray());
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(keyManagerFactory.getKeyManagers(), null, null);
        System.out.println("O SSLContext foi configurado com sucesso.");
    }
}
```

Esse código:

- Carrega o **KeyStore** especificado.
- Verifica se o alias do certificado está presente no KeyStore.
- Configura um **SSLContext** com o certificado carregado para garantir que ele está disponível para ser usado em conexões seguras.

## 3\. Testar a Conexão SSL (Cliente-Servidor)

Você também pode testar diretamente a configuração SSL da JVM em um ambiente servidor (como Tomcat ou Spring Boot) ou cliente, usando uma ferramenta como **curl** ou o **OpenSSL** para testar a conexão segura.

### **Testar com curl**

```shell
curl -v --cacert caminho/para/ca-cert.pem https://localhost:8443
```

Este comando:

- Verifica se o certificado é válido e aceito quando você acessa a aplicação no **localhost** na porta **8443** (ou a porta onde seu servidor SSL está rodando).
- **\--cacert** permite verificar o certificado do servidor em relação ao CA fornecido.

Se tudo estiver correto, a saída mostrará que a conexão foi estabelecida com sucesso e o certificado é aceito.

### **Testar com OpenSSL**

Você pode usar o **OpenSSL** para testar a conexão SSL e verificar os detalhes do certificado:

```shell
openssl s_client -connect localhost:8443
```

A saída desse comando mostrará informações detalhadas sobre o certificado utilizado pelo servidor, como a validade, o emissor, e a cadeia de certificação.

## 4\. Verificar Logs da JVM

Se você estiver usando uma aplicação como **Tomcat** ou **Spring Boot**, verifique os logs de inicialização. Certifique-se de que não haja erros relacionados ao carregamento do KeyStore ou à inicialização do SSL.

Nos logs, você deve ver mensagens indicando que o **SSL** foi inicializado corretamente com o **KeyStore** fornecido.

### **Exemplo de Logs de Inicialização do Spring Boot:**

```shell
Tomcat started on port(s): 8443 (https) with context path ''
```

Se houver erros como "não foi possível carregar o KeyStore" ou "senha incorreta", isso indica que houve algum problema durante o carregamento do certificado.

## 5\. Validar o Certificado com Ferramentas Externas

Se você deseja validar o certificado diretamente, pode exportá-lo do **KeyStore** e usar ferramentas como o **OpenSSL** para inspecionar seus detalhes.

### **Exportar o Certificado com keytool**

```shell
keytool -exportcert -alias seu-alias -file certificado-exportado.crt -keystore caminho/para/keystore.jks -storepass senha-do-keystore
```

Depois, você pode verificar os detalhes do certificado exportado com o **OpenSSL**:

```shell
openssl x509 -in certificado-exportado.crt -text -noout
```

Esse comando mostra os detalhes do certificado, como o emissor, o nome comum (CN), a validade, e a chave pública.

## Conclusão

- Use o **keytool** para listar os certificados no **KeyStore** e verificar se os detalhes estão corretos.
- Teste a conexão SSL com ferramentas como **curl** ou **OpenSSL** para garantir que o certificado está sendo usado corretamente.
- Inspecione os logs da JVM ou configure o SSL programaticamente em Java para validar que o certificado foi carregado corretamente.
- Opcionalmente, exporte o certificado e use o **OpenSSL** para inspecionar seus detalhes.

Esses métodos devem garantir que o certificado está funcionando corretamente dentro da JVM.