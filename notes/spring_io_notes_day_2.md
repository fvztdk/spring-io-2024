# Spring I/O conf 2024 - Day 2

## platform engineering with spring boot

- parent POM with all the dependencies and platform services also a BOM
- multiple spring boot starters with lifecycle, security, mvc, data access, etc
- examples:
    - default properties
        - want to set a default value for ex: in the local environment
        - @PropertySource per environment
    .environment vars translatation
        - read env vars from the platform 
        . use a post processor to read the env vars, and translate to internal model
        - add the post processor to spring.factories file so it's done before auto configuration
    - environment profiles
        - the platform injects the environment name and type and some spring-boot uses this to select profiles
        - uses as well a environment post processor
    - dependecy creation
        - read a resource descriptor and if lets say a redis is needed, a autoconfiguration post processor read the type and auto generate a bean of the type reading the properties like url and username from env vars

## Spring security integration testing

- Integration tests are important because the largest part of our application is dependencies (borrowed code)
- why testing for spring security
    - dependency can change the way it works
    - you test the authorization
- @WithMockUser (authorities='read', username='john')
    - for basic use cases
- managed users
    - programactlly manage how the users are created
    - user details service
    - @WithUserDetails()
    - implement UserDetailService to create you test users
- custom authentication
    - WithSecurityContextFactory
    - manipulate the security context to use the custom authentication implementation
    - create a custom annotation with the custom claim
    - create a custom security context factory


## Password less - passkey
- problems with passwords
    - passwords are knowledge based
    - are prone to phishing
    - remote replay
    - data breach
    - constant change
- alternatives
    - biometric
    - magic links
    - otps
    - push notifications 
- passkeys
    - criptography key-pass that are used to authenticate
    - fido - fast identity online
    - fido2 - current standard
    - webauthn
    - authenticator - ubiqui, other device
    - relying party - request the authentication, hold usernames and public keys
    - client pass authenticaticator data to relying part and check if it matches
    - synced - platform authenticator, app
    - device-based
    - 2 flows
        - registration flow - request challenge and public key with username is stored on the server
        - authentication flow - request a challenge, encode the challenge with private key and send to server for validation 

## event driven autoscaling for java
- horizontal pod autoscaler
    - the problem is that it doesnt work well with event driven application because its based on cpu on memory
    - you might need to scale on some external dependecies, queue size, prometheus metrics, etc that are not inside kubernetes
    - need to create kubernetes custom metrics, but cant be easily connected to a kafka or prometheus
    - only one adapter per kubernetes namespace
- KEDA - kubernetes event driven autoscaling 
    - 64 bult-in scalers 
        - kafka, prometheus, azure monitor, aws cloudwatch
    - application agnostic, use external triggers to controls HPA
    - can be used to make non serveless appplication into serveless (scale to 0)
- Knative
    - apps must be converted to be serveless 
    - push base approach
    - using different autoscaler
    - demand based autoscaling 
- Demos
    - use HPA
        - its slow, on based on internal k8s metrics 
    - use KEDA
        - use kafka trigger to autoscale
        - scales much faster than with HPA
        - no change to code
    - knative
        - needs to be rewrite from kafka consumer to cloudevents consumer
        - redeployed as knative service
        - knative eventing kafka source
        - it's invoked be "source" (kafka)
        - it's serveless, there's no instance until load is detected 
        - knative can't scale to more than one pod
    - knative + keda
        - keda.autoscaling.knative annotation on knative infrastructure
        - needs to install an operator
        - keda scale on kafka sources

## Domain driven design with spring 
- understand the domain
- split big domain into subdomains
- develop an ubiquitus language
- develop a domain model
- separate domain model from implementation

- domain 
    - create record for id, like a generator
    - create record for important information that need validation, like DNI
- domain event listener
    - abstractAgreggateRoot

## Serveless java with Spring
- functions, more serveless than containers
    - simple java method (request event input, Context context) {}
    - runtime version
    - memory, cpu, timeout settings
    - triggered by event
- aws lambda with spring
    - Spring Cloud Function
    - have multiple functions on one codebae
    - aws cloud function invoker routes to specific functions
    - expose functions via http
-  convert @RestControllers to lambda
    - aws serveless java container (library)
- use SnapStart to improve startup times
- additional recommendations
    - use very small connection pools to DBs, has each request will create a connection pool with many connections, or use a proxy to connect to database
    - for observability, use push model, as lambda doesn't expose http ports so you can fetch metrics
    - serverless java replataforming guide

## Advanced testing
-  Junit
    - Parameterized test
        @ValueSource( ints = {10, 13, 14})
        @MethodSource -- return a Stream of Arguments
    - Extensions
       - custom LoggingExtension with a custom extension that implements a LoggingEvent
       - OutputaptureExtension
    - Setup
        - mock mother with a builder pattern 
        - Persons.aPerson().olderThan18().build();
        - within the builder default object with random data
        - Faker library
        - Instancio library
        - Instantiator library
    - Assertions
        - assertj extracting
        - assertThat(result).hasName("...").hasAge(34).has....
        - for equals and hashcode
        - EqualsVerifier
- Spring Boot Testing
    - context caching for @SpringBootTest
    - context test slicing for less thing to be loaded, for integration test of isolated parts
        - @WebMvcTest @DataJpaTest
    - alternatives strategies to spring context
        - Extend
            - create a annotation with a junit extension 
            - Clock to manipulate time inside a test 
            - anotate wiith allow-bean-definition-override
            - @LocalTime ( time = "")
        - Compose
            - @MigrationTest template
            - to test DB migrations before migration and after migration
            - custom annotation
        - Create
            - create a custom context test slice
            - custom annotation
            - @ExtendWith(SpringExtension.class)
            - @BootstrapWith(SpringBootTestContextBootstraper.class)
        - only extend/compose/create as last resort
- Approval testing
- Property based testing
    - @ForAll
    - Arbitraries library
-


    