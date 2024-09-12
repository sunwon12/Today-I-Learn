## **Redis 클라이언트 3가지**

1.  **Jedis**  
    -   쓰레드 세이프하지 않아 잘 쓰이지 않는다.
2.  **Lettuce**
    -   비동기 및 동기식 API를 제공하는 Redis 클라이언트로, Netty 기반으로 구현되어 높은 성능을 자랑합니다.
    -   Redis Cluster와 Sentinel을 기본적으로 지원하며, 스레드 세이프한 구조로 멀티스레드 환경에서도 안정적입니다.
    -   비동기/리액티브 프로그래밍을 위한 API를 제공하므로, 높은 처리량을 필요로 하는 애플리케이션에 적합합니다.
3.  **Redisson**
    -   고급 기능을 제공하는 Redis 클라이언트로, Redis를 분산 데이터 구조로 활용할 수 있게 도와줍니다.
    -   분산 락, 분산 맵, 분산 큐와 같은 기능을 기본적으로 제공하며, 이러한 구조를 사용해 분산 애플리케이션을 개발하는 데 유용합니다.
    -   즉, 분산환경에서 자원 동기화가 필요하면 Redisson을 씀
4.  **Spring Data Redis**  
    -   Jedis나 Lettuce 같은 클라이언트를 내부적으로 사용하여, Spring에서 Redis를 추상화한 고수준 API를 제공합니다.

Jedis는 쓰레드 세이프 하지 않으므로 고려하지 않겠다. 또한 성능 문제가 있다. 자세한 사항이 궁금하다면 [향로님의 글](https://jojoldu.tistory.com/418)을 확인해봐라.

레디스 클라이언트를 고를 때 기준은 이러하다.

분산환경에서 자원 동기화를 원한다면 Redisson를 ,

큰 특이사항이 없다면 스프링이 설정한 기본 클라이언트인 Lettuce를 사용하면 된다.

---

## **Redis Lettuce**

![image](https://github.com/user-attachments/assets/6129784f-cb62-41fe-b2f5-4e30f20d94ba)

Lettuce는 Netty 기반 라이브러리로 Asynchronous & Non-blocking으로 구현되어 있다. 

여기서 비동기 방식이 핵심이다. 비동기 방식은 싱크를 맞추는 것과 상관없음. 즉, 하나의 작업이 완료될 때까지 대기하지 않고 다른 작업을 처리할 수 있는 특징이 있다. 이러한 비동기 방식으로 동작하는 Lettuce는 Redis와 하나의 커넥션만으로 여러 요청을 비동기적으로 처리할 수 있기에, 리소스를 효율적으로 사용하고 응답시간을 최적화한다.

 spring boot 2부터는 레터스가 기본이기 때문에 자동 설정에 의해 Redis Connection factory로 레터스가 사용된다. 스프링부트에서는 일반적으로 RedisTemplate추상화 레벨을 이용하므로 lettuce api를 직접 호출할 일은 없다.

비동기로 Redis 요청을 보낼 때에는 connection pool이 필요가 없다. 동기화로 Redis 요청을 보내고 **동시 요청이 많을 때** 사용할 수 있는 Redis 연결의 개수를 늘려야 하기 때문에 connection pool을 조정할 필요가 있다.

connection pool은 애플리케이션의 쓰레드를 Redis와 연결할 파이프라인이라고 생각하면 된다. 쓰레드가 4개이고 pool이 한 개라면 순서대로 앞에 쓰레드가 명령을 보낼 때까지 기다려야 한다.

---

## **+) 레터스 커넥션 풀**

전용 커넥션을 획득하기 위해 매번 커넥션을 획득.반환하게 되면 비용이 많이 발생하기 때문에 커넥션 풀을 사용하여 비용을 절감할 수 있다.

스프링부트에서 레터스 커넥션 풀을 위한 옵션을 제공한다.

```
spring.redis.lettuce.pool.max-active
spring.redis.lettuce.pool.max-idle
spring.redis.lettuce.pool.max-wait
spring.redis.lettuce.pool.min-idle
spring.redis.lettuce.pool.time-between-eviction-runs
```

lettuce.pool 옵션이 있으면 커넥션 풀이 생성되며, 전용 커넥션이 필요할 때 커넥션 풀이 사용된다.

레터스 커넥션 풀을 사용하려면 apache common-pool2이 필요하므로 pom.xml에 dependency를 추가해야 합니다.

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.4.3</version>
</dependency>
```

_(LettuceConnectionFactory @bean이 있으면 자동 설정에 의해 LettuceConnectionFactory이 생성되지 않으니 위 옵션들은 당연히 먹히지 않습니다. 혹시 모르니....)_

---

## **RedisTemplate(동기화) vs ReactiveRedisTemplate(비동기화)**

1.  **RedisTemplate**:
    -   RedisTemplate은 Spring Data Redis에서 제공하는 동기화된 API입니다.
    -   Lettuce 클라이언트를 사용하더라도, RedisTemplate은 동기적으로 작동하도록 설계되어 있습니다. 이 API는 내부적으로 블로킹 I/O 작업을 수행하기 때문에 호출하는 쪽에서는 요청을 기다리며 결과를 받아오게 됩니다.
    -   즉, Lettuce가 비동기적으로 요청을 보내고 받더라도, RedisTemplate은 이를 동기적으로 감싸기 때문에 클라이언트는 동기화된 응답을 받게 됩니다.
2.  **ReactiveRedisTemplate**:
    -   ReactiveRedisTemplate은 비동기 및 논블로킹 방식으로 동작하는 API입니다.
    -   ReactiveRedisTemplate은 Lettuce의 비동기적이고 논블로킹 방식과 잘 맞물려 Mono나 Flux와 같은 **리액티브 프로그래밍 모델**을 통해 데이터를 주고받습니다.
    -   이 경우, 요청이 바로 리턴되고, 결과는 나중에 이벤트로 처리됩니다. 따라서 결과를 기다리지 않고도 다른 작업을 비동기적으로 처리할 수 있습니다.

**RedisTemplate를 위한 config**

```
@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private String redisPort;



    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(redisHost);
        redisStandaloneConfiguration.setPort(Integer.parseInt(redisPort));
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
        return lettuceConnectionFactory;
    }
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }
}
```

**ReactiveRedisTemplate****를 위한 config**

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.data.redis.connection.ReactiveRedisConnectionFactory;
import org.springframework.data.redis.core.ReactiveRedisTemplate;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import reactor.core.publisher.Mono;

@Configuration
public class RedisConfig {

    @Value("${spring.data.redis.host}")
    private String redisHost;

    @Value("${spring.data.redis.port}")
    private String redisPort;

    // 비동기 작업을 위한 ReactiveRedisConnectionFactory
    @Bean
    public ReactiveRedisConnectionFactory reactiveRedisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(redisHost);
        redisStandaloneConfiguration.setPort(Integer.parseInt(redisPort));
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    // 비동기 작업을 위한 ReactiveRedisTemplate
    @Bean
    public ReactiveRedisTemplate<String, Object> reactiveRedisTemplate(ReactiveRedisConnectionFactory factory) {
        return new ReactiveRedisTemplate<>(factory, RedisSerializationContext.string());
    }
}





    // 비동기 작업 예시
    @Autowired
    private ReactiveRedisTemplate<String, Object> reactiveRedisTemplate;

    public Mono<Boolean> setValueAsync(String key, Object value) {
        return reactiveRedisTemplate.opsForValue().set(key, value);
    }

    public Mono<Object> getValueAsync(String key) {
        return reactiveRedisTemplate.opsForValue().get(key);
    }
}
```
