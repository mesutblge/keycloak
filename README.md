# Keycloak Kurulumu ve Spring Boot Entegrasyonu

Bu belge, Keycloak kimlik ve erişim yönetimi sağlayıcısını nasıl kuracağınızı ve Spring Boot uygulamanızla nasıl entegre edeceğinizi adım adım açıklar.

## Keycloak Kurulumu

1. Projemizin bulunduğu klasörde YAML uzantılı dosyamızın olduğu klasöre gidin.
2. Terminalde `docker-compose up -d` komutunu çalıştırarak container'ı ayağa kaldırın.
3. Eğer container başarıyla ayağa kalktıktan sonra kullanıcı adı ve şifre hatası alırsanız başka bir terminalde aşağıdaki kodu çalıştırarak Keycloak'u başlatın:

    ```bash
    docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:24.0.1 start-dev
    ```

4. Kodun çalışmasıyla birlikte Keycloak, 8080 portundan erişilebilir hale gelecektir.
5. Tarayıcıda `http://localhost:8080` adresine giderek Keycloak yönetim paneline erişin.
6. Kullanıcı adı `admin` ve şifre `admin` ile giriş yapın.

## Keycloak Ayarları

1. Keycloak yönetim panelinde "Create realm" butonuna tıklayarak bir realm oluşturun.
![Create Realm Ekran Görüntüsü](/home/b044/Desktop/createrealm.png)

2. Users alanına girip Add User diyerek yeni bir user oluşturun.
![Create User Ekran Görüntüsü](/home/b044/Desktop/createuser.png)

3. "Clients" menüsüne girip "Create client" ile yeni bir client oluşturun.
![Create Client Ekran Görüntüsü](/home/b044/Desktop/createclient.png)
3.a " General Settings " alanında Client ID kısmını dolduruyoruz.
![Client ID Ekran Görüntüsü](/home/b044/Desktop/generalsettings.png)
3.b " Capability Config " alanında Client authentication butonunu aktif hale getiriyoruz.
![Client Authentication Ekran Görüntüsü](/home/b044/Desktop/capabilityconfig.png)
3.c " Login Settings " bölümünde "Valid redirect URLs" alanına projenizin çalıştığı URL'yi yazın ve "Web origins" alanında Cross origin belirtin.* işareti koyarak tüm makinelerin bağlanabileceğini belirtiyoruz.
![Login Settings Ekran Görüntüsü](/home/b044/Desktop/loginsettings.png)

4. Save dedikten sonra Credentials'a girip Client Secret alanını kopyalıyoruz.
![Client Secret Ekran Görüntüsü](/home/b044/Desktop/clientsecret.png)



## Spring Boot Entegrasyonu

1. Spring Boot projesinde `application.properties` dosyasını açın ve aşağıdaki bilgileri ekleyin:

    ```properties
    ## Keycloak
    spring.security.oauth2.client.provider.Demo.issuer-uri=http://localhost:8080/realms/Demo
    spring.security.oauth2.client.registration.Demo.provider=Demo
    spring.security.oauth2.client.registration.Demo.client-id=Demo-Client
    spring.security.oauth2.client.registration.Demo.client-secret=(kopyaladiginiz_secret_keyi_buraya_yapistirin)
    spring.security.oauth2.client.registration.Demo.scope=openid,offline_access,profile
    spring.security.oauth2.client.registration.Demo.authorization-grant-type=authorization_code
    ```

2. Projeyi çalıştırmak için aşağıdaki bağımlılıkları `pom.xml` dosyasına ekleyin:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

```

3. Spring Boot projenize `Keycloak` adında bir controller dosyası oluşturun ve aşağıdaki kodları ekleyin:

    ```java
    import jakarta.servlet.http.HttpServletRequest;
    import org.springframework.security.core.Authentication;
    import org.springframework.security.core.context.SecurityContextHolder;
    import org.springframework.security.oauth2.core.user.OAuth2User;
    import org.springframework.security.web.authentication.logout.SecurityContextLogoutHandler;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RestController;
        
    import java.util.HashMap;
        
    @RestController
    public class Keycloak {
        @GetMapping(path = "/")
        public HashMap index() {
            // get a successful user login
            OAuth2User user = ((OAuth2User) SecurityContextHolder.getContext().getAuthentication().getPrincipal());
            return new HashMap(){{
                put(user.getAttribute("name")," sisteme giriş yaptı.");
            }};
        }
        
        @GetMapping(path = "/unauthenticated")
        public HashMap unauthenticatedRequests(HttpServletRequest request) {
            // Logout the current user
            Authentication auth = SecurityContextHolder.getContext().getAuthentication();
            if (auth != null) {
                new SecurityContextLogoutHandler().logout(request, null, auth);
            }
        
            return new HashMap(){{
                put("message", "Logged out of the system.");
            }};
        }
    }
    ```

3. Projeyi çalıştırdıktan sonra `localhost:8094` adresine istek attığınızda Keycloak login sayfası karşılayacaktır. Kullanıcı adı ve şifreyi girdikten sonra bilgilere erişebilirsiniz.

## Yardımcı API

Keycloak hakkında daha fazla bilgi için [resmi dokümantasyonu](https://www.keycloak.org/docs/latest) ziyaret edebilirsiniz.
