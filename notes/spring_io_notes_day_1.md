# Spring I/O conf 2024 - Day 1

## opening 

1200 persons, 59 countries

84 speakers, 59 talks


## keynote
### spring update and roadmap

spring boot 3.3
- cds support for daster jvm startup
- observability improvements


spring boot 3.4
- deep core container SpEL revision
- fallback bean, bean override in tests
- release november 2024


### runtime efficiency 3.3

- *spring executable jar self extract*
- *CDS support in buildpacks*
- *42% percent  reduce startup time*
- *35% reduce memory footprint*
- *200% + performance improvements with spring virtual threads flag*
- MockMvcTester assertJ

### spring modulith

 *what it does*

- verify arch and code
    - built in arch unit
    - can test modules independently
- visualize your system and arch
    - auto generate drawns of arch
    - auto zipkin to track interaction between modules

- integration
    - integration through events to decouple modules simplified with annotations
    - support for republish of events in case of fails
    - @Externalize to externalize events to external event bus

- open application modules
    - legay code and modularized code can cohexist
    - open class to execute old code while migrating 


### Spring AI 
 
- spring ai loaders
- to integrate with ai platforms
- vector store to expose database to AI in a friendly format


## Java news
- yeld
- decomposed
- unnamed variable in decompose

## Java virtual threads 
- Continuation API (don't use in your code)
- copy a stack part from/to the heap/stack

## Dapr
- DAPR (distributable appplication resources)
- injects a sidecar with some apis the expose other resources in the cliuster
- the dapr api is an abstraction to be dependecy agnostic (http/grpc)
- in local:
    - import dapr docker container
    - app just call daprClient
    - key/value store
    - pubsub
    - MessasingTemplate
    - KeyValueTemplate
    - Outbox pattern
        - send a message when data stored (transactionally)
        - dapr can do it for you, you don't need to call both dependecies


## Spring Core - Juergen Hoeller
- Autowiring Algorithm
    - Autowiring by Type+Qualifier
        - by type
        - by qualifiers, only in conjuction with type
        - for non-unique, by primary
        - if still not unique, by name
    - In 6.2 now first it match by name, *use for better performance* 
    - Fallback beans - @Fallback
        - if theres at least one non fallback it will be used, otherwise the fallback will be used
        - it's the contrary of @Primary
- Container Initialization
    - Singleton Locking 
        - singleton initialization lock
        - on startup; initialize all if not explicity @Lazy
        - all the locking happen in a single thread
        - In 6.2 use of ReentrantLock on the main singleton thread so other threads use lenient lock to be aware of the lock
        - Avoid full deadlock during initialization
    - Background Initialization Options
        - lenient locking makes it feasible to use multiple threads
        - multiple threads lead to less predictable startup behaviour
        - RECOMENDATION TO STILL USE A SINGLE THREAD
        - JPA bootstrapping on different thread
        - in 6.2 you can @Bean(bootstrap=BACKGROUND) to background initialization of any @Bean; to be used with @Lazy injection points

## GraalVM - Going AOT
- what is: is a JDK with ahead of time
- you can do AOT or JIT
- available since spring boot 3
- slow to compile
- how it compiles:
    - tree shaking, only gets what is needed
    - run initializations
    - look at the heap and optimizes
- Key points:
    - lower resource usage
    - starts super fast
    - predictable performance (all the performance just after start)
    - extra security layer over standard JVM (allow list for dependencies during compilation, not JIT compiler attacks)
    - compact images
    - support from many frameworks and cloud
- spring aot
    - assist during compile
        - java source files
        - bytecode
        - hint files
- libraries for native
    - graalVM can do reflection, but libraries must support native
    - must support config options
    - can configure custom hints for custom library
    - graalvm reachbility metadata (repo with configs to make libraries work)
    - ready for native image
    - tracing agent to produce necessary config automatically
- demos
    - use to create a CLI command (spring shell)
- performance
    - with no extra parameters is slower
    - with some tuning it can be faster, with some analyzing of the running code to better optimize
- best practices
    - cross platform builds on github actions
    - build container images with graalVm and buildpacks
    - reduced attack surface
- recommendations 
    - photo
    - modern-java-demo - github - nikolay
- what's next
    - layered native images
        - fast recompilation
        - resource sharing during deployment
    - oracle cloud - graalOS, run the binaries directly

## Migrate from JPA to Spring Data JDBC
- recommendations for migration ?
    - it's easy for a small project, but in big projects you might have to do a lot of changes
    - you should have very good persistence layer tests before starting the migration
    - start with baby steps
    - mikado method - for refactory
    - sd jdbc uses aggregates - is a block of connected classes, sd jdbc only do operations on the aggregate root level (insert, fetch)
    - breadth-first or depth-first, migrate all the aggregates first
- migration steps:
    - identify aggregates - things that can't exist without the other
    - remove all the uncessary repositories that is duplicated when thinking about the agreggates; refactor to use the father aggregate to manipulate child classes
    - make save statments explicits
    - remove cascade between aggregates
    - simplify bidirectional references to unidirectional references
        - from what comes first to later
        - within aggregates, from father to childs
    - replace java class references by ids
    - add the dependecy to the project
    - create an IdGenerator, JPA does that, JDBC expects the DB to do it or you need to do it
    - create read/write converters (mappers)
    - migrate one aggregate at a time (check the correct imports are used persistence vs data)
