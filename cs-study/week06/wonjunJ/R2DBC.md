## JDBC

- **작동원리**
JDBC는 데이터베이스 요청을 처리할 때, 클라이언트 측 쓰레드가 데이터베이스 작업을 요청하고 그 응답을 기다리는 동안 해당 쓰레드가 블록된다. 이러한 방식은 동기적이며, 각 작업이 순차적으로 완료되기를 기다리는 방식으로 운영되기에 블록킹 방식이다.

## R2DBC

- R2DBC는 interface를 제공하는 Specification 일 뿐이다. 데이터베이스 벤더 측에서 해당 데이터베이스에 맞도록 구현해야만 한다.
- R2DBC는 Reactive streams API를 기반으로 만들어졌다.
- **작동원리**
R2DBC(Reactive Relational Database Connectivity)는 리액티브 프로그래밍을 지원하는 새로운 형태의 데이터베이스 연결 드라이버로, 논블록킹 방식을 채택하고 있다. 클라이언트 측에서 데이터베이스 요청을 보내면, R2DBC는 이 요청을 비동기적으로 처리하며, 이는 클라이언트 측의 쓰레드가 데이터베이스 서버로부터 응답을 기다리는 동안에도 다른 작업을 계속해서 수행할 수 있도록 한다. 따라서 R2DBC를 사용할 때, 논블록킹 처리는 데이터베이스 서버의 쓰레드가 아니라, DB를 호출하는 웹 서버 또는 애플리케이션 서버의 클라이언트 측 쓰레드에서 발생한다.

## JDBC VS R2DBC

- **R2DBC와 JPA와의 호환성**
    - JPA는 블록킹 방식의 데이터베이스 작업을 가정하고 설계된 기술로, 각 데이터베이스 작업이 완료될 때까지 스레드를 대기 상태로 유지한다.
    - 반면, R2DBC는 대기 없이 비동기 처리를 지향하기 때문에, JPA와 함께 사용할 경우 성능 저하나 디자인 상의 제약을 초래할 수 있다.
- **차이점**
    
    ![Untitled](./difference.png)
    
- **WebMVC, WebFlux에서 각 조합에 대한 비교**
    
    ![Untitled](./comparison.png)
    
    WebFlux라고 무조건 높은 concurrency를 보장하는건 아니다. R2DBC 덕분에 높은 concurrency 시에 빠른 응답 시간을 가진다.
    
    ### 요약
    
    R2DBC + Webflux가 높은 동시성때는 좋은 선택
    
    - 요청당 CPU를 조금 사용함
    - 요청당 메모리가 조금 필요함
    - 응답시간이 빠름
    - 처리량도 높아짐
    - JAR도 가벼워짐
    - 하지만, 대략 200 concurrenct request 밑으로는 Spring Web MVC + JDBC가 좋은 선택
- **현재, R2DBC를 지원하는 관계형 DB**
    - H2
    - MySQL
    - jasync-sql (MySQL를 사용하기 위해 대신 사용하는 것, 공식적으로 제공하는 건 없음)
    - MariaDB
    - Microsoft SQL Server
    - PostgreSQL
    - OracleDB
    - Cloud Spanner(Google Cloud Platform)

## R2DBC 사용하기

- 2가지의 중요한 인터페이스
    - **ConnectionFactory** - io.r2dbc.spi package
        - 말 그래도 데이터베이스 드라이버와 Connection을 생성하는 인터페이스이다.
        - 드라이버 구현체에서 이를 구현해서 사용하게 된다. Jasync-sql을 사용하면 JasyncConnectionFactory 클래스가 구현체로 사용된다.
    - **DatabaseClient** - org.springframework.r2dbc.core package
        - ConnectionFactory 를 사용하는 non-blocking, reactive client이다.
        - 사용 예시
            
            ```java
            DatabaseClient client = DatabaseClient.create(factory);
            Mono<Actor> actor = client.sql("select first_name, last_name from t_actor")
                .map(row -> new Actor(row.get("first_name", String.class),
                     row.get("last_name", String.class)))
                .first();
            ```
            
    - 이러한 것들의 대부분의 구현체들을 Spring auto configuration을 통해 자동으로 bean으로 등록해준다.
- yml파일 설정
    
    ```yaml
    r2dbc:
        url: "r2dbc:mariadb://root:0722@localhost:3307/flupet_service" #url에 db의 아이디와 비번 함께 담아서 설정
      #    username: [db아이디] # 이렇게 설정하니까 오류 발생
      #    password: [비번]
      
    # 추가 설정 - R2DBC에서 Database로 나가는 쿼리를 보고 싶으면 아래처럼 로깅레벨을 설정해야 함
    logging:
      level:
        org.springframework.r2dbc.core: debug
    ```
    
- R2dbc Audit 기능
    - R2dbc audit은 @CreatedBy, @CreatedDate, @LastModifiedBy, @LastModifiedDate annotation을 활성화시켜준다.
    - Configuration class 만들어서 설정
        
        ```kotlin
        @Configuration
        @EnableR2dbcAuditing
        class R2dbcConfig
        ```
        
- 각 테이블에 대한 Entity class 매핑
    
    ```kotlin
    //코틀린에서는 lombok 사용할 수 없음
    @Table("member_flupet")
    data class MemberFlupet(
        @Id
        val id: Long? = null,
        @Column("flupet_id")
        var flupetId: Long? = null,
        @Column("member_id")
        var memberId: String? = null,
        var name: String,
        var exp: Int = 0,
        var steps: Long = 0,
        @Column("is_dead")
        var isDead: Boolean = false,
        //태어난 날짜
        @CreatedDate
        @Column("create_time")
        var createTime: LocalDateTime? = null,
    ) {
    }
    ```
    
    - Spring data r2dbc는 Relational Mapping을 지원하지 않음
    - Join이 필요한 상황이라면 직접 Query를 작성(밑에서 설명)
- 쉽게 CRUD를 사용할 수 있도록 Repository interface를 제공
    
    ```kotlin
    @Repository
    interface MemberFlupetRepository: ReactiveCrudRepository<MemberFlupet, Long>, MemberFlupetRepositoryCustom {
        fun findAllByIsDeadIsFalse(): Flux<MemberFlupet> //다수의 데이터를 리턴한다면 Flux
        fun findByMemberIdAndIsDeadIsFalse(memberId: String): Mono<MemberFlupet> //단건의 데이터를 리턴한다면 Mono
        fun findAllByMemberIdAndIsDeadIsTrue(memberId: String): Flux<MemberFlupet>
        fun existsByMemberIdAndIsDeadIsFalse(memberId: String): Mono<Boolean>
    }
    ```
    
    - 만약 코틀린(코루틴)을 사용한다면 별도로 제공
        
        ```kotlin
        @Repository
        interface FlupetRepository: CoroutineCrudRepository<Flupet, Long> {
            fun findByStage(stage: Int): Flow<Flupet> //코루틴 환경에서 Flux와 매핑 되는 것
        }
        ```
        
        ```kotlin
        //Mono → suspend 
        fun handler(): Mono<Void> -> suspend fun handler()
        
        //Flux → Flow
        fun handler(): Flux<T> -> fun handler(): Flow<T>
        
        //Flow로 반환된 값을 사용하려면 collect를 사용(이거를 호출하지 않으면 아무일도 일어나지 않는다.)
        flow.collect { value -> println(value) }
        ```
        
- **참고**
    - Spring data R2DBC의 작동 방식
        - save() 메소드는 들어오는 엔티티의 기본키의 값이 null 일 경우에 새로운 엔티티로 판단해 Insert 쿼리를 실행하고, null 이 아닐 경우에 Update 쿼리를 실행한다.
        - 이로 인해서 ID 필드에 Auto Increment를 설정하지 않았을시 문제가 발생할 수 있다.
            - 별도의 insert문을 작성해서 사용해야 함

## JOOQ

- Spring Data R2DBC에서는 Join을 사용할 수 없음
    - Query 어노테이션을 활용하여 join을 할 수는 있다.
- 일반적으로 JPA환경에서 많이 사용하는 QueryDSL은 비동기 논블로킹 방식을 지원하지 않으므로 R2DBC에서는 사용할 수 없음
- 최근, JOOQ에서 R2DBC에서 사용할 수 있도록 논블로킹 방식을 지원하기 시작하면서 사용할 수 있음
- **사용 방법**
    - JOOQ는 Spring에서 Configuration class를 통해서 사용 설정이 안되는듯
    - QueryDSL인 경우 자동으로 Q클래스가 생성이 되지만, JOOQ는 자동으로 저러한 클래스가 생성이 되지 않으므로 별도로 설정 필요
    - gradle파일에서 설정을 할 수 있음
        
        ```groovy
        //application.yml에서 정보를 가져오기 위해 설정
        def loadYaml(String path) {
            def yaml = new Yaml()
            def inputStream = new FileInputStream(new File(path))
            def data = yaml.load(inputStream) as Map
            return data
        }
        def application = loadYaml('src/main/resources/application.yml')
        
        println "DB URL: ${System.getenv('DB_URL')}"
        
        jooq {
            version = '3.17.5' //3.15버전 부터 논블로킹 방식 사용 가능
            edition = nu.studer.gradle.jooq.JooqEdition.OSS
        
            configurations {
                create("main") {
        		        //자동으로 Q클래스와 같은 것을 생성을 할 것인가에 true로 설정
                    generateSchemaSourceOnCompilation = true
        
                    generationTool {
                        logging = org.jooq.meta.jaxb.Logging.WARN
                        jdbc {
                            driver = 'org.mariadb.jdbc.Driver'
                            url = System.getenv('DB_URL')
                            user = System.getenv('DB_USER')
                            password = System.getenv('DB_PASSWORD')
                            //properties.add(new Property().withKey('ssl').withValue('false'))
                        }
                        generator {
                            database {
                                name = 'org.jooq.meta.mariadb.MariaDBDatabase' // Updated for MariaDB
                                inputSchema = 'fluffit_flupet' // Ensure this matches your MariaDB schema
                            }
                            target {
                                packageName = 'com.ssafy.jooq'
                                directory = 'src/main/generated'
                            }
                            strategy {
                                name = 'org.jooq.codegen.DefaultGeneratorStrategy'
                            }
                        }
                    }
                }
            }
        }
        ```
        
    - Query문 작성
        - DSLContext를 사용하기 위한 설정
            
            ```kotlin
            @Configuration
            class DslConfig(
                val factory: ConnectionFactory
            ) {
                @Bean
                fun dslContext() = DSL.using(factory)
            }
            ```
            
        - 실제 쿼리문
            
            ```kotlin
            override suspend fun findMainInfoByUserId(userId: String): MainInfoDto? {
                //return null
                return withContext(Dispatchers.IO) {
                    Mono.from(
                        dslContext
                            .select(
                                MEMBER_FLUPET.FULLNESS.`as`("fullness"),
                                MEMBER_FLUPET.HEALTH.`as`("health"),
                                MEMBER_FLUPET.NAME.`as`("flupetName")
                            )
                            .from(MEMBER_FLUPET)
                            .join(FLUPET)
                            .on(MEMBER_FLUPET.FLUPET_ID.eq(FLUPET.ID))
                            .where(MEMBER_FLUPET.MEMBER_ID.eq(userId))
                            .orderBy(
                                MEMBER_FLUPET.END_TIME.isNull.desc(),
                                MEMBER_FLUPET.END_TIME.desc()
                            )
                            .limit(1)
                    )
                        .map { record -> record.into(MainInfoDto::class.java) }
                        .awaitFirstOrNull()
                }
            }
            
            override fun findFlupetsByUserId(userId: String): Flow<CollectionResponse.Flupet> {
                return Flux.from(
                    dslContext
                        .select(
                            FLUPET.NAME.`as`("species"),
                            FLUPET.IMG_URL,
                            FLUPET.STAGE.`as`("tier"),
                            DSL.field( //jooq에서 임의의 새로운 필드를 생성하기 위한 방법
                                DSL.exists(
                                    dslContext
                                        .selectOne()
                                        .from(MEMBER_FLUPET)
                                        .where(
                                            MEMBER_FLUPET.FLUPET_ID.eq(FLUPET.ID)
                                                .and(MEMBER_FLUPET.MEMBER_ID.eq(userId))
                                        )
                                )
                            ).`as`("metBefore")
                        )
                        .from(FLUPET)
                        .orderBy(
                            FLUPET.ID.asc()
                        )
                )
                    .map { record ->
                        CollectionResponse.Flupet(
                            species = record.get("species", String::class.java),
                            imageUrl = record.get(FLUPET.IMG_URL, String::class.java).split(","),
                            tier = record.get("tier", Int::class.java),
                            metBefore = record.get("metBefore", Boolean::class.java)
                        )
                    }
                    .asFlow()
                //return dslContext.selectOne().fetchInto(CollectionResponse.Flupet::class.java).asFlow()
            }
            ```
            

### 참고

- [https://mindybughunter.com/spring-webflux2편-r2dbc/](https://mindybughunter.com/spring-webflux2%ED%8E%B8-r2dbc/)
- [https://happyer16.tistory.com/entry/스프링-blocking-vs-non-blocking-R2DBC-vs-JDBC-WebFlux-vs-Web-MVC](https://happyer16.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81-blocking-vs-non-blocking-R2DBC-vs-JDBC-WebFlux-vs-Web-MVC)
- https://binux.tistory.com/155
- [https://devsh.tistory.com/entry/스프링-웹플럭스와-코루틴-톺아보기](https://devsh.tistory.com/entry/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9B%B9%ED%94%8C%EB%9F%AD%EC%8A%A4%EC%99%80-%EC%BD%94%EB%A3%A8%ED%8B%B4-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0)
