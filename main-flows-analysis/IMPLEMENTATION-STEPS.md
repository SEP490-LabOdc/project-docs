# Các bước triển khai chi tiết cho từng Microservice

Tài liệu này cung cấp hướng dẫn từng bước để triển khai các microservice trong hệ thống Lab-based ODC Platform. Mỗi service sẽ được triển khai theo một quy trình chuẩn, đảm bảo tính nhất quán và dễ bảo trì.

## Quy trình triển khai chung

### 1. Thiết lập dự án

1. Tạo dự án Spring Boot mới sử dụng Spring Initializr (https://start.spring.io/)
2. Chọn các dependency cần thiết:
   - Spring Web
   - Spring Data JPA
   - Spring Security (nếu cần)
   - Spring Cloud (nếu cần)
   - Database connector (PostgreSQL, MongoDB)
   - Các dependency đặc thù cho service
3. Tải về và giải nén dự án
4. Import dự án vào IDE (IntelliJ IDEA, Eclipse, VS Code)

### 2. Cấu hình cơ bản

1. Cấu hình `application.yml` với các thông tin:
   - Cổng server
   - Kết nối database
   - Cấu hình logging
   - Các cấu hình đặc thù cho service
2. Cấu hình `bootstrap.yml` (nếu sử dụng Spring Cloud Config):
   - Tên ứng dụng
   - Thông tin kết nối đến Config Server
3. Tạo các package cơ bản:
   - `config`: Chứa các lớp cấu hình
   - `controller`: Chứa các REST controller
   - `dto`: Chứa các Data Transfer Object
   - `exception`: Chứa các lớp xử lý ngoại lệ
   - `model`: Chứa các entity
   - `repository`: Chứa các repository
   - `service`: Chứa các service interface và implementation

### 3. Triển khai các lớp cơ bản

1. Tạo các entity trong package `model`
2. Tạo các repository trong package `repository`
3. Tạo các DTO trong package `dto`
4. Tạo các service interface và implementation trong package `service`
5. Tạo các controller trong package `controller`
6. Tạo các lớp xử lý ngoại lệ trong package `exception`

### 4. Triển khai tính năng đặc thù

1. Triển khai các tính năng đặc thù cho từng service
2. Tích hợp với các service khác (nếu cần)
3. Triển khai các tính năng bảo mật (nếu cần)

### 5. Kiểm thử

1. Viết unit test cho các service và repository
2. Viết integration test cho các controller
3. Kiểm thử API bằng Postman hoặc Swagger

### 6. Đóng gói và triển khai

1. Build dự án thành file JAR
2. Tạo Dockerfile
3. Build Docker image
4. Triển khai lên Kubernetes hoặc môi trường khác

## Triển khai chi tiết cho từng service

### 1. User Service

#### Bước 1: Thiết lập dự án

```bash
# Tạo dự án Spring Boot mới
mkdir -p user-service
cd user-service

# Tạo cấu trúc thư mục
mkdir -p src/main/java/com/odc/user/config
mkdir -p src/main/java/com/odc/user/controller
mkdir -p src/main/java/com/odc/user/dto
mkdir -p src/main/java/com/odc/user/exception
mkdir -p src/main/java/com/odc/user/model
mkdir -p src/main/java/com/odc/user/repository
mkdir -p src/main/java/com/odc/user/service
mkdir -p src/main/resources
mkdir -p src/test/java/com/odc/user
```

#### Bước 2: Cấu hình cơ bản

1. Tạo file `pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/>
    </parent>
    <groupId>com.odc</groupId>
    <artifactId>user-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user-service</name>
    <description>User Service for ODC Platform</description>

    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

2. Tạo file `application.yml`:

```yaml
server:
  port: 8081

spring:
  application:
    name: user-service
  datasource:
    url: jdbc:postgresql://localhost:5432/odc_user
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect

jwt:
  secret: odcPlatformSecretKey
  expiration: 86400000

logging:
  level:
    com.odc.user: DEBUG
    org.springframework: INFO
    org.hibernate: INFO
```

3. Tạo file `bootstrap.yml`:

```yaml
spring:
  application:
    name: user-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
```

#### Bước 3: Triển khai các lớp cơ bản

1. Tạo lớp chính của ứng dụng:

```java
package com.odc.user;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

2. Tạo các entity:

```java
package com.odc.user.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    @Column(nullable = false, unique = true)
    private String email;

    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String fullName;

    private String phone;

    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();

    private boolean enabled = true;
}
```

```java
package com.odc.user.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Entity
@Table(name = "roles")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private ERole name;

    public enum ERole {
        ROLE_ADMIN,
        ROLE_BUSINESS,
        ROLE_TALENT,
        ROLE_MENTOR
    }
}
```

3. Tạo các repository:

```java
package com.odc.user.repository;

import com.odc.user.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    Boolean existsByUsername(String username);
    Boolean existsByEmail(String email);
}
```

```java
package com.odc.user.repository;

import com.odc.user.model.Role;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface RoleRepository extends JpaRepository<Role, Long> {
    Optional<Role> findByName(Role.ERole name);
}
```

4. Tạo các DTO:

```java
package com.odc.user.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.constraints.Email;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Size;
import java.util.Set;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserDTO {
    private Long id;

    @NotBlank
    @Size(min = 3, max = 20)
    private String username;

    @NotBlank
    @Size(max = 50)
    @Email
    private String email;

    @NotBlank
    @Size(min = 6, max = 40)
    private String password;

    @NotBlank
    private String fullName;

    private String phone;

    private Set<String> roles;
}
```

```java
package com.odc.user.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.validation.constraints.NotBlank;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AuthRequestDTO {
    @NotBlank
    private String username;

    @NotBlank
    private String password;
}
```

```java
package com.odc.user.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AuthResponseDTO {
    private String token;
    private String type = "Bearer";
    private Long id;
    private String username;
    private String email;
    private List<String> roles;

    public AuthResponseDTO(String token, Long id, String username, String email, List<String> roles) {
        this.token = token;
        this.id = id;
        this.username = username;
        this.email = email;
        this.roles = roles;
    }
}
```

5. Tạo các service:

```java
package com.odc.user.service;

import com.odc.user.dto.UserDTO;
import com.odc.user.model.User;

import java.util.List;

public interface UserService {
    User createUser(UserDTO userDTO);
    User updateUser(Long id, UserDTO userDTO);
    User getUserById(Long id);
    User getUserByUsername(String username);
    List<User> getAllUsers();
    void deleteUser(Long id);
}
```

```java
package com.odc.user.service.impl;

import com.odc.user.dto.UserDTO;
import com.odc.user.exception.ResourceNotFoundException;
import com.odc.user.model.Role;
import com.odc.user.model.User;
import com.odc.user.repository.RoleRepository;
import com.odc.user.repository.UserRepository;
import com.odc.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RoleRepository roleRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Override
    public User createUser(UserDTO userDTO) {
        User user = new User();
        user.setUsername(userDTO.getUsername());
        user.setEmail(userDTO.getEmail());
        user.setPassword(passwordEncoder.encode(userDTO.getPassword()));
        user.setFullName(userDTO.getFullName());
        user.setPhone(userDTO.getPhone());

        Set<Role> roles = new HashSet<>();
        if (userDTO.getRoles() == null || userDTO.getRoles().isEmpty()) {
            Role userRole = roleRepository.findByName(Role.ERole.ROLE_TALENT)
                    .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
            roles.add(userRole);
        } else {
            userDTO.getRoles().forEach(role -> {
                switch (role) {
                    case "admin":
                        Role adminRole = roleRepository.findByName(Role.ERole.ROLE_ADMIN)
                                .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
                        roles.add(adminRole);
                        break;
                    case "business":
                        Role businessRole = roleRepository.findByName(Role.ERole.ROLE_BUSINESS)
                                .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
                        roles.add(businessRole);
                        break;
                    case "mentor":
                        Role mentorRole = roleRepository.findByName(Role.ERole.ROLE_MENTOR)
                                .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
                        roles.add(mentorRole);
                        break;
                    default:
                        Role talentRole = roleRepository.findByName(Role.ERole.ROLE_TALENT)
                                .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
                        roles.add(talentRole);
                }
            });
        }

        user.setRoles(roles);
        return userRepository.save(user);
    }

    @Override
    public User updateUser(Long id, UserDTO userDTO) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));

        user.setFullName(userDTO.getFullName());
        user.setPhone(userDTO.getPhone());

        if (userDTO.getPassword() != null && !userDTO.getPassword().isEmpty()) {
            user.setPassword(passwordEncoder.encode(userDTO.getPassword()));
        }

        return userRepository.save(user);
    }

    @Override
    public User getUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
    }

    @Override
    public User getUserByUsername(String username) {
        return userRepository.findByUsername(username)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with username: " + username));
    }

    @Override
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @Override
    public void deleteUser(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found with id: " + id));
        userRepository.delete(user);
    }
}
```

6. Tạo các controller:

```java
package com.odc.user.controller;

import com.odc.user.dto.UserDTO;
import com.odc.user.model.User;
import com.odc.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import javax.validation.Valid;
import java.util.List;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping
    public ResponseEntity<User> createUser(@Valid @RequestBody UserDTO userDTO) {
        User user = userService.createUser(userDTO);
        return new ResponseEntity<>(user, HttpStatus.CREATED);
    }

    @GetMapping("/{id}")
    public ResponseEntity<User> getUserById(@PathVariable Long id) {
        User user = userService.getUserById(id);
        return ResponseEntity.ok(user);
    }

    @GetMapping
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return ResponseEntity.ok(users);
    }

    @PutMapping("/{id}")
    public ResponseEntity<User> updateUser(@PathVariable Long id, @Valid @RequestBody UserDTO userDTO) {
        User updatedUser = userService.updateUser(id, userDTO);
        return ResponseEntity.ok(updatedUser);
    }

    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

```java
package com.odc.user.controller;

import com.odc.user.dto.AuthRequestDTO;
import com.odc.user.dto.AuthResponseDTO;
import com.odc.user.dto.UserDTO;
import com.odc.user.model.User;
import com.odc.user.security.jwt.JwtUtils;
import com.odc.user.security.services.UserDetailsImpl;
import com.odc.user.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.Valid;
import java.util.List;
import java.util.stream.Collectors;

@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserService userService;

    @Autowired
    private JwtUtils jwtUtils;

    @PostMapping("/signin")
    public ResponseEntity<AuthResponseDTO> authenticateUser(@Valid @RequestBody AuthRequestDTO loginRequest) {
        Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword()));

        SecurityContextHolder.getContext().setAuthentication(authentication);
        String jwt = jwtUtils.generateJwtToken(authentication);

        UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();
        List<String> roles = userDetails.getAuthorities().stream()
                .map(item -> item.getAuthority())
                .collect(Collectors.toList());

        return ResponseEntity.ok(new AuthResponseDTO(
                jwt,
                userDetails.getId(),
                userDetails.getUsername(),
                userDetails.getEmail(),
                roles));
    }

    @PostMapping("/signup")
    public ResponseEntity<User> registerUser(@Valid @RequestBody UserDTO signUpRequest) {
        User user = userService.createUser(signUpRequest);
        return ResponseEntity.ok(user);
    }
}
```

7. Tạo các lớp xử lý ngoại lệ:

```java
package com.odc.user.exception;

import org.springframework.http.HttpStatus;
import org.springframework.web.bind.annotation.ResponseStatus;

@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}
```

```java
package com.odc.user.exception;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.context.request.WebRequest;

import java.util.Date;
import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorDetails> handleResourceNotFoundException(
            ResourceNotFoundException exception, WebRequest webRequest) {
        ErrorDetails errorDetails = new ErrorDetails(new Date(), exception.getMessage(),
                webRequest.getDescription(false));
        return new ResponseEntity<>(errorDetails, HttpStatus.NOT_FOUND);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorDetails> handleGlobalException(
            Exception exception, WebRequest webRequest) {
        ErrorDetails errorDetails = new ErrorDetails(new Date(), exception.getMessage(),
                webRequest.getDescription(false));
        return new ResponseEntity<>(errorDetails, HttpStatus.INTERNAL_SERVER_ERROR);
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Object> handleMethodArgumentNotValidException(
            MethodArgumentNotValidException exception) {
        Map<String, String> errors = new HashMap<>();
        exception.getBindingResult().getAllErrors().forEach((error) -> {
            String fieldName = ((FieldError) error).getField();
            String errorMessage = error.getDefaultMessage();
            errors.put(fieldName, errorMessage);
        });
        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

```java
package com.odc.user.exception;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Date;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ErrorDetails {
    private Date timestamp;
    private String message;
    private String details;
}
```

#### Bước 4: Triển khai tính năng bảo mật

1. Tạo các lớp cấu hình bảo mật:

```java
package com.odc.user.security;

import com.odc.user.security.jwt.AuthEntryPointJwt;
import com.odc.user.security.jwt.AuthTokenFilter;
import com.odc.user.security.services.UserDetailsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    UserDetailsServiceImpl userDetailsService;

    @Autowired
    private AuthEntryPointJwt unauthorizedHandler;

    @Bean
    public AuthTokenFilter authenticationJwtTokenFilter() {
        return new AuthTokenFilter();
    }

    @Override
    public void configure(AuthenticationManagerBuilder authenticationManagerBuilder) throws Exception {
        authenticationManagerBuilder.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and().csrf().disable()
                .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                .authorizeRequests()
                .antMatchers("/api/auth/**").permitAll()
                .antMatchers("/api/test/**").permitAll()
                .anyRequest().authenticated();

        http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
    }
}
```

2. Tạo các lớp JWT:

```java
package com.odc.user.security.jwt;

import com.odc.user.security.services.UserDetailsImpl;
import io.jsonwebtoken.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Component;

import java.util.Date;

@Component
public class JwtUtils {
    private static final Logger logger = LoggerFactory.getLogger(JwtUtils.class);

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration}")
    private int jwtExpirationMs;

    public String generateJwtToken(Authentication authentication) {
        UserDetailsImpl userPrincipal = (UserDetailsImpl) authentication.getPrincipal();

        return Jwts.builder()
                .setSubject((userPrincipal.getUsername()))
                .setIssuedAt(new Date())
                .setExpiration(new Date((new Date()).getTime() + jwtExpirationMs))
                .signWith(SignatureAlgorithm.HS512, jwtSecret)
                .compact();
    }

    public String getUserNameFromJwtToken(String token) {
        return Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token).getBody().getSubject();
    }

    public boolean validateJwtToken(String authToken) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
            return true;
        } catch (SignatureException e) {
            logger.error("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            logger.error("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            logger.error("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            logger.error("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            logger.error("JWT claims string is empty: {}", e.getMessage());
        }

        return false;
    }
}
```

```java
package com.odc.user.security.jwt;

import com.odc.user.security.services.UserDetailsServiceImpl;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.util.StringUtils;
import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class AuthTokenFilter extends OncePerRequestFilter {

    @Autowired
    private JwtUtils jwtUtils;

    @Autowired
    private UserDetailsServiceImpl userDetailsService;

    private static final Logger logger = LoggerFactory.getLogger(AuthTokenFilter.class);

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String jwt = parseJwt(request);
            if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
                String username = jwtUtils.getUserNameFromJwtToken(jwt);

                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception e) {
            logger.error("Cannot set user authentication: {}", e);
        }

        filterChain.doFilter(request, response);
    }

    private String parseJwt(HttpServletRequest request) {
        String headerAuth = request.getHeader("Authorization");

        if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
            return headerAuth.substring(7);
        }

        return null;
    }
}
```

```java
package com.odc.user.security.jwt;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.AuthenticationEntryPoint;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class AuthEntryPointJwt implements AuthenticationEntryPoint {

    private static final Logger logger = LoggerFactory.getLogger(AuthEntryPointJwt.class);

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authException) throws IOException, ServletException {
        logger.error("Unauthorized error: {}", authException.getMessage());
        response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Error: Unauthorized");
    }
}
```

3. Tạo các lớp UserDetails:

```java
package com.odc.user.security.services;

import com.fasterxml.jackson.annotation.JsonIgnore;
import com.odc.user.model.User;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.Collection;
import java.util.List;
import java.util.Objects;
import java.util.stream.Collectors;

public class UserDetailsImpl implements UserDetails {
    private static final long serialVersionUID = 1L;

    private Long id;
    private String username;
    private String email;
    @JsonIgnore
    private String password;
    private Collection<? extends GrantedAuthority> authorities;

    public UserDetailsImpl(Long id, String username, String email, String password,
                           Collection<? extends GrantedAuthority> authorities) {
        this.id = id;
        this.username = username;
        this.email = email;
        this.password = password;
        this.authorities = authorities;
    }

    public static UserDetailsImpl build(User user) {
        List<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority(role.getName().name()))
                .collect(Collectors.toList());

        return new UserDetailsImpl(
                user.getId(),
                user.getUsername(),
                user.getEmail(),
                user.getPassword(),
                authorities);
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    public Long getId() {
        return id;
    }

    public String getEmail() {
        return email;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;
        UserDetailsImpl user = (UserDetailsImpl) o;
        return Objects.equals(id, user.id);
    }
}
```

```java
package com.odc.user.security.services;

import com.odc.user.model.User;
import com.odc.user.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    UserRepository userRepository;

    @Override
    @Transactional
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User Not Found with username: " + username));

        return UserDetailsImpl.build(user);
    }
}
```

#### Bước 5: Kiểm thử

1. Tạo các unit test cho service:

```java
package com.odc.user.service;

import com.odc.user.dto.UserDTO;
import com.odc.user.model.Role;
import com.odc.user.model.User;
import com.odc.user.repository.RoleRepository;
import com.odc.user.repository.UserRepository;
import com.odc.user.service.impl.UserServiceImpl;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.security.crypto.password.PasswordEncoder;

import java.util.HashSet;
import java.util.Optional;
import java.util.Set;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;

@ExtendWith(MockitoExtension.class)
public class UserServiceTests {

    @Mock
    private UserRepository userRepository;

    @Mock
    private RoleRepository roleRepository;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserServiceImpl userService;

    private User user;
    private UserDTO userDTO;
    private Role role;

    @BeforeEach
    public void setup() {
        role = new Role();
        role.setId(1L);
        role.setName(Role.ERole.ROLE_TALENT);

        Set<Role> roles = new HashSet<>();
        roles.add(role);

        user = new User();
        user.setId(1L);
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("password");
        user.setFullName("Test User");
        user.setPhone("1234567890");
        user.setRoles(roles);

        Set<String> roleStrings = new HashSet<>();
        roleStrings.add("talent");

        userDTO = new UserDTO();
        userDTO.setUsername("testuser");
        userDTO.setEmail("test@example.com");
        userDTO.setPassword("password");
        userDTO.setFullName("Test User");
        userDTO.setPhone("1234567890");
        userDTO.setRoles(roleStrings);
    }

    @Test
    public void givenUserDTO_whenCreateUser_thenReturnUser() {
        // given
        given(roleRepository.findByName(Role.ERole.ROLE_TALENT)).willReturn(Optional.of(role));
        given(passwordEncoder.encode(userDTO.getPassword())).willReturn("encodedPassword");
        given(userRepository.save(any(User.class))).willReturn(user);

        // when
        User savedUser = userService.createUser(userDTO);

        // then
        assertThat(savedUser).isNotNull();
        assertThat(savedUser.getUsername()).isEqualTo(userDTO.getUsername());
        verify(userRepository, times(1)).save(any(User.class));
    }

    @Test
    public void givenUserId_whenGetUserById_thenReturnUser() {
        // given
        given(userRepository.findById(1L)).willReturn(Optional.of(user));

        // when
        User foundUser = userService.getUserById(1L);

        // then
        assertThat(foundUser).isNotNull();
        assertThat(foundUser.getId()).isEqualTo(1L);
    }
}
```

2. Tạo các integration test cho controller:

```java
package com.odc.user.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.odc.user.dto.UserDTO;
import com.odc.user.model.User;
import com.odc.user.service.UserService;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.web.servlet.MockMvc;

import java.util.HashSet;
import java.util.Set;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
public class UserControllerTests {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private UserService userService;

    @Autowired
    private ObjectMapper objectMapper;

    private User user;
    private UserDTO userDTO;

    @BeforeEach
    public void setup() {
        user = new User();
        user.setId(1L);
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        user.setPassword("password");
        user.setFullName("Test User");
        user.setPhone("1234567890");

        Set<String> roles = new HashSet<>();
        roles.add("talent");

        userDTO = new UserDTO();
        userDTO.setUsername("testuser");
        userDTO.setEmail("test@example.com");
        userDTO.setPassword("password");
        userDTO.setFullName("Test User");
        userDTO.setPhone("1234567890");
        userDTO.setRoles(roles);
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    public void givenUserDTO_whenCreateUser_thenReturn201() throws Exception {
        // given
        given(userService.createUser(any(UserDTO.class))).willReturn(user);

        // when & then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userDTO)))
                .andExpect(status().isCreated())
                .andExpect(jsonPath("$.id").value(user.getId()))
                .andExpect(jsonPath("$.username").value(user.getUsername()));
    }

    @Test
    @WithMockUser
    public void givenUserId_whenGetUserById_thenReturnUser() throws Exception {
        // given
        given(userService.getUserById(1L)).willReturn(user);

        // when & then
        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(user.getId()))
                .andExpect(jsonPath("$.username").value(user.getUsername()));
    }

    @Test
    @WithMockUser
    public void givenUserDTO_whenUpdateUser_thenReturnUpdatedUser() throws Exception {
        // given
        given(userService.updateUser(eq(1L), any(UserDTO.class))).willReturn(user);

        // when & then
        mockMvc.perform(put("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(userDTO)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(user.getId()))
                .andExpect(jsonPath("$.username").value(user.getUsername()));
    }
}
```

#### Bước 6: Đóng gói và triển khai

1. Tạo Dockerfile:

```dockerfile
FROM openjdk:11-jdk-slim
VOLUME /tmp
COPY target/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

2. Tạo docker-compose.yml:

```yaml
version: '3'

services:
  postgres:
    image: postgres:13
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: odc_user
    volumes:
      - postgres_data:/var/lib/postgresql/data

  user-service:
    build: .
    ports:
      - "8081:8081"
    depends_on:
      - postgres
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/odc_user
      SPRING_DATASOURCE_USERNAME: postgres
      SPRING_DATASOURCE_PASSWORD: postgres

volumes:
  postgres_data:
```

3. Tạo Kubernetes deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  labels:
    app: user-service
spec:
  replicas: 1
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: odc/user-service:latest
        ports:
        - containerPort: 8081
        env:
        - name: SPRING_DATASOURCE_URL
          value: jdbc:postgresql://postgres:5432/odc_user
        - name: SPRING_DATASOURCE_USERNAME
          value: postgres
        - name: SPRING_DATASOURCE_PASSWORD
          value: postgres
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
spec:
  selector:
    app: user-service
  ports:
  - port: 8081
    targetPort: 8081
  type: ClusterIP
```

### 2. Analytics Service

#### Bước 1: Thiết lập dự án

```bash
# Tạo dự án Spring Boot mới
mkdir -p analytics-service
cd analytics-service

# Tạo cấu trúc thư mục
mkdir -p src/main/java/com/odc/analytics/config
mkdir -p src/main/java/com/odc/analytics/controller
mkdir -p src/main/java/com/odc/analytics/dto
mkdir -p src/main/java/com/odc/analytics/exception
mkdir -p src/main/java/com/odc/analytics/model
mkdir -p src/main/java/com/odc/analytics/repository
mkdir -p src/main/java/com/odc/analytics/service
mkdir -p src/main/resources
mkdir -p src/test/java/com/odc/analytics
```

#### Bước 2: Cấu hình cơ bản

1. Tạo file `pom.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.0</version>
        <relativePath/>
    </parent>
    <groupId>com.odc</groupId>
    <artifactId>analytics-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>analytics-service</name>
    <description>Analytics Service for ODC Platform</description>

    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
        <spark.version>3.3.0</spark.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

2. Tạo file `application.yml`:

```yaml
server:
  port: 8085

spring:
  application:
    name: analytics-service
  datasource:
    url: jdbc:postgresql://localhost:5432/odc_analytics
    username: postgres
    password: postgres
    driver-class-name: org.postgresql.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
  redis:
    host: localhost
    port: 6379

spark:
  app-name: odc-analytics
  master: local[*]

logging:
  level:
    com.odc.analytics: DEBUG
    org.springframework: INFO
    org.hibernate: INFO
```

3. Tạo file `bootstrap.yml`:

```yaml
spring:
  application:
    name: analytics-service
  cloud:
    config:
      uri: http://localhost:8888
      fail-fast: true
```

#### Bước 3: Triển khai các lớp cơ bản

1. Tạo lớp chính của ứng dụng:

```java
package com.odc.analytics;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients
public class AnalyticsServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(AnalyticsServiceApplication.class, args);
    }
}
```

2. Tạo các entity:

```java
package com.odc.analytics.controller;

import com.odc.analytics.dto.AnalyticsRequestDTO;
import com.odc.analytics.dto.ReportDTO;
import com.odc.analytics.service.ReportService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/analytics/reports")
public class ReportController {

    @Autowired
    private ReportService reportService;

    @GetMapping
    public ResponseEntity<List<ReportDTO>> getAllReports() {
        return ResponseEntity.ok(reportService.getAllReports());
    }

    @GetMapping("/{id}")
    public ResponseEntity<ReportDTO> getReportById(@PathVariable Long id) {
        return ResponseEntity.ok(reportService.getReportById(id));
    }

    @GetMapping("/type/{type}")
    public ResponseEntity<List<ReportDTO>> getReportsByType(@PathVariable String type) {
        return ResponseEntity.ok(reportService.getReportsByType(type));
    }

    @PostMapping
    public ResponseEntity<ReportDTO> createReport(@RequestBody ReportDTO reportDTO) {
        return new ResponseEntity<>(reportService.createReport(reportDTO), HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    public ResponseEntity<ReportDTO> updateReport(
            @PathVariable Long id,
            @RequestBody ReportDTO reportDTO) {
        return ResponseEntity.ok(reportService.updateReport(id, reportDTO));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteReport(@PathVariable Long id) {
        reportService.deleteReport(id);
        return ResponseEntity.noContent().build();
    }

    @PostMapping("/generate")
    public ResponseEntity<ReportDTO> generateReport(@RequestBody AnalyticsRequestDTO requestDTO) {
        return new ResponseEntity<>(reportService.generateReport(requestDTO), HttpStatus.CREATED);
    }
}
```

7. Cấu hình Apache Spark:

```java
package com.odc.analytics.config;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaSparkContext;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SparkConfig {

    @Value("${spark.app.name:analytics-service}")
    private String appName;

    @Value("${spark.master:local[*]}")
    private String masterUri;

    @Bean
    public SparkConf sparkConf() {
        return new SparkConf()
                .setAppName(appName)
                .setMaster(masterUri);
    }

    @Bean
    public JavaSparkContext javaSparkContext() {
        return new JavaSparkContext(sparkConf());
    }
}
```

8. Cấu hình Redis:

```java
package com.odc.analytics.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class RedisConfig {

    @Value("${spring.redis.host:localhost}")
    private String redisHost;

    @Value("${spring.redis.port:6379}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration(redisHost, redisPort);
        return new LettuceConnectionFactory(config);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new Jackson2JsonRedisSerializer<>(Object.class));
        return template;
    }
}
```

9. Tạo lớp tiện ích để xử lý dữ liệu với Spark:

```java
package com.odc.analytics.util;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Component
public class SparkDataProcessor {

    @Autowired
    private JavaSparkContext sparkContext;

    /**
     * Phân tích dữ liệu dự án để tạo báo cáo
     * 
     * @param projectData Dữ liệu dự án dưới dạng danh sách các bản ghi
     * @param parameters Các tham số phân tích
     * @return Kết quả phân tích dưới dạng chuỗi JSON
     */
    public String analyzeProjectData(List<Map<String, Object>> projectData, Map<String, Object> parameters) {
        // Tạo RDD từ dữ liệu dự án
        JavaRDD<Map<String, Object>> projectRDD = sparkContext.parallelize(projectData);
        
        // Tạo SparkSession để sử dụng Spark SQL
        SparkSession spark = SparkSession.builder()
                .appName("Project Data Analysis")
                .getOrCreate();
        
        // Chuyển đổi RDD thành DataFrame
        Dataset<Row> projectDF = spark.createDataFrame(projectRDD, Map.class);
        
        // Đăng ký DataFrame như một bảng tạm thời
        projectDF.createOrReplaceTempView("projects");
        
        // Thực hiện phân tích dựa trên tham số
        String timeRange = parameters.getOrDefault("timeRange", "last30days").toString();
        String groupBy = parameters.getOrDefault("groupBy", "status").toString();
        
        // Xây dựng truy vấn SQL dựa trên tham số
        String sqlQuery = buildAnalyticsQuery(timeRange, groupBy);
        
        // Thực thi truy vấn
        Dataset<Row> resultDF = spark.sql(sqlQuery);
        
        // Chuyển đổi kết quả thành JSON
        String resultJson = resultDF.toJSON().collectAsList().toString();
        
        return resultJson;
    }
    
    /**
     * Xây dựng truy vấn SQL dựa trên tham số
     */
    private String buildAnalyticsQuery(String timeRange, String groupBy) {
        StringBuilder query = new StringBuilder();
        query.append("SELECT ").append(groupBy).append(", COUNT(*) as count ");
        query.append("FROM projects ");
        
        // Thêm điều kiện thời gian
        if ("last30days".equals(timeRange)) {
            query.append("WHERE created_date >= date_sub(current_date(), 30) ");
        } else if ("last90days".equals(timeRange)) {
            query.append("WHERE created_date >= date_sub(current_date(), 90) ");
        } else if ("lastYear".equals(timeRange)) {
            query.append("WHERE created_date >= date_sub(current_date(), 365) ");
        }
        
        // Nhóm theo trường được chỉ định
        query.append("GROUP BY ").append(groupBy).append(" ");
        query.append("ORDER BY count DESC");
        
        return query.toString();
    }
}
```

10. Tạo lớp tiện ích để tích hợp với các service khác:

```java
package com.odc.analytics.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

import java.util.List;
import java.util.Map;

@FeignClient(name = "project-service")
public interface ProjectServiceClient {

    @GetMapping("/api/projects/data")
    List<Map<String, Object>> getProjectsData();
    
    @GetMapping("/api/projects/{id}/metrics")
    Map<String, Object> getProjectMetrics(@PathVariable("id") Long projectId);
}
```

```java
package com.odc.analytics.client;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;

import java.util.List;
import java.util.Map;

@FeignClient(name = "talent-pool-service")
public interface TalentPoolServiceClient {

    @GetMapping("/api/talents/data")
    List<Map<String, Object>> getTalentsData();
    
    @GetMapping("/api/talents/skills/statistics")
    Map<String, Object> getSkillsStatistics();
}
```

11. Cấu hình Dockerfile:

```dockerfile
FROM openjdk:11-jdk-slim

WORKDIR /app

COPY target/analytics-service-0.0.1-SNAPSHOT.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

12. Cấu hình docker-compose.yml:

```yaml
version: '3.8'

services:
  analytics-service:
    build: .
    ports:
      - "8085:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=docker
      - SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/analytics_db
      - SPRING_DATASOURCE_USERNAME=postgres
      - SPRING_DATASOURCE_PASSWORD=postgres
      - SPRING_REDIS_HOST=redis
      - SPRING_REDIS_PORT=6379
      - EUREKA_CLIENT_SERVICEURL_DEFAULTZONE=http://eureka-server:8761/eureka/
    depends_on:
      - postgres
      - redis
      - eureka-server

  postgres:
    image: postgres:13
    ports:
      - "5435:5432"
    environment:
      - POSTGRES_DB=analytics_db
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:6
    ports:
      - "6379:6379"

  eureka-server:
    image: odc/eureka-server:latest
    ports:
      - "8761:8761"

volumes:
  postgres-data:
```

## Triển khai các service khác

Các service khác trong hệ thống như User Service, Talent Pool Service, Business Service, ODC Team Service, Project Service, Skill Matching Service, Marketplace Service, Learning Service, và Notification Service cũng sẽ được triển khai theo cấu trúc tương tự như đã trình bày ở trên với User Service và Analytics Service.

Mỗi service sẽ có cấu trúc cơ bản như sau:

1. Cấu trúc thư mục chuẩn Spring Boot
2. Cấu hình pom.xml với các dependency phù hợp
3. Cấu hình application.yml và bootstrap.yml
4. Lớp ứng dụng chính với các annotation cần thiết
5. Các entity tương ứng với bảng trong cơ sở dữ liệu
6. Các repository để truy cập dữ liệu
7. Các DTO để truyền dữ liệu
8. Các service interface và implementation
9. Các controller để xử lý các request API
10. Các lớp cấu hình bổ sung (nếu cần)
11. Các lớp tiện ích và client để tích hợp với các service khác
12. Cấu hình Dockerfile và docker-compose.yml

Việc triển khai chi tiết cho từng service sẽ phụ thuộc vào yêu cầu cụ thể của service đó, nhưng cấu trúc cơ bản sẽ giống nhau.

## Kết luận

Tài liệu này đã trình bày chi tiết các bước triển khai microservices cho hệ thống Lab-based ODC Platform, bao gồm:

1. Quy trình triển khai chung cho các microservice
2. Triển khai chi tiết cho User Service
3. Triển khai chi tiết cho Analytics Service
4. Hướng dẫn triển khai cho các service khác

Việc tuân thủ các hướng dẫn này sẽ giúp đảm bảo tính nhất quán trong việc triển khai các microservice, giúp quá trình phát triển và bảo trì hệ thống trở nên dễ dàng hơn.
```
```

```java
package com.odc.analytics.repository;

import com.odc.analytics.model.Dashboard;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface DashboardRepository extends JpaRepository<Dashboard, Long> {
}
```

4. Tạo các DTO:

```java
package com.odc.analytics.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ReportDTO {
    private Long id;
    private String name;
    private String type;
    private String description;
    private String parameters;
    private String resultData;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

```java
package com.odc.analytics.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class DashboardDTO {
    private Long id;
    private String name;
    private String description;
    private String configuration;
    private String data;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
}
```

```java
package com.odc.analytics.dto;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Map;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class AnalyticsRequestDTO {
    private String reportType;
    private Map<String, Object> parameters;
    private String timeRange;
    private String groupBy;
}
```

5. Tạo các service:

```java
package com.odc.analytics.service;

import com.odc.analytics.dto.DashboardDTO;

import java.util.List;

public interface DashboardService {
    List<DashboardDTO> getAllDashboards();
    DashboardDTO getDashboardById(Long id);
    DashboardDTO createDashboard(DashboardDTO dashboardDTO);
    DashboardDTO updateDashboard(Long id, DashboardDTO dashboardDTO);
    void deleteDashboard(Long id);
}
```

```java
package com.odc.analytics.service;

import com.odc.analytics.dto.ReportDTO;
import com.odc.analytics.dto.AnalyticsRequestDTO;

import java.util.List;

public interface ReportService {
    List<ReportDTO> getAllReports();
    ReportDTO getReportById(Long id);
    List<ReportDTO> getReportsByType(String type);
    ReportDTO createReport(ReportDTO reportDTO);
    ReportDTO updateReport(Long id, ReportDTO reportDTO);
    void deleteReport(Long id);
    ReportDTO generateReport(AnalyticsRequestDTO requestDTO);
}
```

```java
package com.odc.analytics.service.impl;

import com.odc.analytics.dto.DashboardDTO;
import com.odc.analytics.model.Dashboard;
import com.odc.analytics.repository.DashboardRepository;
import com.odc.analytics.service.DashboardService;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class DashboardServiceImpl implements DashboardService {

    @Autowired
    private DashboardRepository dashboardRepository;

    @Autowired
    private ModelMapper modelMapper;

    @Override
    public List<DashboardDTO> getAllDashboards() {
        return dashboardRepository.findAll().stream()
                .map(dashboard -> modelMapper.map(dashboard, DashboardDTO.class))
                .collect(Collectors.toList());
    }

    @Override
    public DashboardDTO getDashboardById(Long id) {
        Dashboard dashboard = dashboardRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Dashboard not found with id: " + id));
        return modelMapper.map(dashboard, DashboardDTO.class);
    }

    @Override
    public DashboardDTO createDashboard(DashboardDTO dashboardDTO) {
        Dashboard dashboard = modelMapper.map(dashboardDTO, Dashboard.class);
        Dashboard savedDashboard = dashboardRepository.save(dashboard);
        return modelMapper.map(savedDashboard, DashboardDTO.class);
    }

    @Override
    public DashboardDTO updateDashboard(Long id, DashboardDTO dashboardDTO) {
        Dashboard existingDashboard = dashboardRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Dashboard not found with id: " + id));
        
        existingDashboard.setName(dashboardDTO.getName());
        existingDashboard.setDescription(dashboardDTO.getDescription());
        existingDashboard.setConfiguration(dashboardDTO.getConfiguration());
        existingDashboard.setData(dashboardDTO.getData());
        
        Dashboard updatedDashboard = dashboardRepository.save(existingDashboard);
        return modelMapper.map(updatedDashboard, DashboardDTO.class);
    }

    @Override
    public void deleteDashboard(Long id) {
        Dashboard dashboard = dashboardRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Dashboard not found with id: " + id));
        dashboardRepository.delete(dashboard);
    }
}
```

```java
package com.odc.analytics.service.impl;

import com.odc.analytics.dto.AnalyticsRequestDTO;
import com.odc.analytics.dto.ReportDTO;
import com.odc.analytics.model.Report;
import com.odc.analytics.repository.ReportRepository;
import com.odc.analytics.service.ReportService;
import org.modelmapper.ModelMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class ReportServiceImpl implements ReportService {

    @Autowired
    private ReportRepository reportRepository;

    @Autowired
    private ModelMapper modelMapper;

    @Override
    public List<ReportDTO> getAllReports() {
        return reportRepository.findAll().stream()
                .map(report -> modelMapper.map(report, ReportDTO.class))
                .collect(Collectors.toList());
    }

    @Override
    public ReportDTO getReportById(Long id) {
        Report report = reportRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Report not found with id: " + id));
        return modelMapper.map(report, ReportDTO.class);
    }

    @Override
    public List<ReportDTO> getReportsByType(String type) {
        return reportRepository.findByType(type).stream()
                .map(report -> modelMapper.map(report, ReportDTO.class))
                .collect(Collectors.toList());
    }

    @Override
    public ReportDTO createReport(ReportDTO reportDTO) {
        Report report = modelMapper.map(reportDTO, Report.class);
        Report savedReport = reportRepository.save(report);
        return modelMapper.map(savedReport, ReportDTO.class);
    }

    @Override
    public ReportDTO updateReport(Long id, ReportDTO reportDTO) {
        Report existingReport = reportRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Report not found with id: " + id));
        
        existingReport.setName(reportDTO.getName());
        existingReport.setType(reportDTO.getType());
        existingReport.setDescription(reportDTO.getDescription());
        existingReport.setParameters(reportDTO.getParameters());
        existingReport.setResultData(reportDTO.getResultData());
        
        Report updatedReport = reportRepository.save(existingReport);
        return modelMapper.map(updatedReport, ReportDTO.class);
    }

    @Override
    public void deleteReport(Long id) {
        Report report = reportRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Report not found with id: " + id));
        reportRepository.delete(report);
    }

    @Override
    public ReportDTO generateReport(AnalyticsRequestDTO requestDTO) {
        // Đây là nơi để triển khai logic phân tích dữ liệu thực tế
        // Có thể sử dụng Apache Spark để xử lý dữ liệu lớn
        // Ví dụ đơn giản:
        Report report = new Report();
        report.setName("Generated Report - " + requestDTO.getReportType());
        report.setType(requestDTO.getReportType());
        report.setDescription("Auto-generated report based on request");
        report.setParameters(requestDTO.getParameters().toString());
        
        // Giả lập dữ liệu kết quả - trong thực tế sẽ là kết quả phân tích
        String resultData = "{\
            \"data\": [\
                {\"category\": \"Category A\", \"value\": 25},\
                {\"category\": \"Category B\", \"value\": 40},\
                {\"category\": \"Category C\", \"value\": 35}\
            ],\
            \"timeRange\": \"" + requestDTO.getTimeRange() + "\",\
            \"groupBy\": \"" + requestDTO.getGroupBy() + "\"\
        }";
        
        report.setResultData(resultData);
        
        Report savedReport = reportRepository.save(report);
        return modelMapper.map(savedReport, ReportDTO.class);
    }
}
```

6. Tạo các controller:

```java
package com.odc.analytics.controller;

import com.odc.analytics.dto.DashboardDTO;
import com.odc.analytics.service.DashboardService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/analytics/dashboard")
public class DashboardController {

    @Autowired
    private DashboardService dashboardService;

    @GetMapping
    public ResponseEntity<List<DashboardDTO>> getAllDashboards() {
        return ResponseEntity.ok(dashboardService.getAllDashboards());
    }

    @GetMapping("/{id}")
    public ResponseEntity<DashboardDTO> getDashboardById(@PathVariable Long id) {
        return ResponseEntity.ok(dashboardService.getDashboardById(id));
    }

    @PostMapping
    public ResponseEntity<DashboardDTO> createDashboard(@RequestBody DashboardDTO dashboardDTO) {
        return new ResponseEntity<>(dashboardService.createDashboard(dashboardDTO), HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    public ResponseEntity<DashboardDTO> updateDashboard(
            @PathVariable Long id,
            @RequestBody DashboardDTO dashboardDTO) {
        return ResponseEntity.ok(dashboardService.updateDashboard(id, dashboardDTO));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteDashboard(@PathVariable Long id) {
        dashboardService.deleteDashboard(id);
        return ResponseEntity.noContent().build();
    }
}
```

```java
package com.odc.analytics.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "dashboards")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Dashboard {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(columnDefinition = "TEXT")
    private String description;

    @Column(columnDefinition = "TEXT")
    private String configuration;

    @Column(columnDefinition = "TEXT")
    private String data;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```