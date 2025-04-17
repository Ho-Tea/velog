<blockquote>
<h3 id="오늘의-목표">오늘의 목표</h3>
<p>스프링에서는 어떻게 <code>@Transactional</code> 하나로 트랜잭션이 적용될 수 있는지 전반적인 과정에 대해 알아보겠습니다.</p>
</blockquote>
<p><a href="https://velog.io/@ho-tea/%EC%9E%90%EB%8F%99%EA%B5%AC%EC%84%B1..%EA%B7%BC%EB%8D%B0-%EC%9D%B4%EC%A0%9C-Data-JPA%EB%A5%BC-%EA%B3%81%EB%93%A4%EC%9D%B8">자동구성..근데-이제-Data-JPA를-곁들인</a> 을 통해 <code>Data JPA</code>를 사용하게 되었을 때 어떠한 자동구성이 일어나게 되는지를 확인하면서 마지막에 <code>TransactionAutoConfiguration</code>을 잠깐 다루었습니다. 
<code>TransactionAutoConfiguration</code>을 통해 트랜잭션 관리를 위한 AOP 설정 및 인프라가 구성이 되는데, 해당 글은 <code>TransactionAutoConfiguration</code>에서부터 시작해 스프링에서 어떻게 <code>@Transactional</code> 하나로 트랜잭션이 적용될 수 있는지 전반적인 과정에 대해 알아보겠습니다.</p>
<h3 id="transactionautoconfiguration">TransactionAutoConfiguration</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/88c9af17-f637-46e9-a520-d17c1bad5d3c/image.png" /></p>
<p><code>TransactionAutoConfiguration</code>의 내부 코드를 자세히 살펴보면 아래와 같이 구성되어 있습니다.</p>
<pre><code class="language-java">// 트랜잭션 자동 구성 클래스
@AutoConfiguration // Spring Boot의 자동 구성 클래스임을 선언
@ConditionalOnClass(PlatformTransactionManager.class) // Classpath에 트랜잭션 관련 클래스가 있을 경우에만 활성화
public class TransactionAutoConfiguration {

    /**
     * 리액티브 트랜잭션 처리용 빈 등록
     * - ReactiveTransactionManager가 하나만 있을 경우, TransactionalOperator 빈을 자동 등록
     */
    @Bean
    @ConditionalOnMissingBean // 이미 동일한 타입의 빈이 없는 경우에만 등록
    @ConditionalOnSingleCandidate(ReactiveTransactionManager.class) // 후보가 하나일 경우에만 등록
    public TransactionalOperator transactionalOperator(ReactiveTransactionManager transactionManager) {
        return TransactionalOperator.create(transactionManager);
    }

    /**
     * 일반 트랜잭션을 위한 TransactionTemplate 구성
     * - PlatformTransactionManager가 단일 후보로 있을 경우 적용
     */
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnSingleCandidate(PlatformTransactionManager.class)
    public static class TransactionTemplateConfiguration {

        /**
         * TransactionOperations 빈이 없는 경우 TransactionTemplate 등록
         * - 명시적 트랜잭션 처리 시 사용됨
         */
        @Bean
        @ConditionalOnMissingBean(TransactionOperations.class)
        public TransactionTemplate transactionTemplate(PlatformTransactionManager transactionManager) {
            return new TransactionTemplate(transactionManager);
        }

    }

    /**
     * 트랜잭션 AOP (프록시 기반 @Transactional) 활성화 구성
     * - 트랜잭션 매니저가 있고, 수동으로 @EnableTransactionManagement를 안 붙였을 경우에만 적용
     */
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnBean(TransactionManager.class) // 트랜잭션 매니저가 있을 때만 적용
    @ConditionalOnMissingBean(AbstractTransactionManagementConfiguration.class) // 개발자가 수동 설정하지 않았을 경우
    public static class EnableTransactionManagementConfiguration {

        /**
         * JDK 동적 프록시 방식 (@EnableTransactionManagement(proxyTargetClass = false))
         * - 인터페이스 기반 프록시
         * - spring.aop.proxy-target-class=false일 경우 적용
         */
        @Configuration(proxyBeanMethods = false)
        @EnableTransactionManagement(proxyTargetClass = false)
        @ConditionalOnProperty(prefix = &quot;spring.aop&quot;, name = &quot;proxy-target-class&quot;, havingValue = &quot;false&quot;)
        public static class JdkDynamicAutoProxyConfiguration {
            // 설정용 빈만 존재, 실제 로직은 없음
        }

        /**
         * CGLIB 기반 클래스 프록시 (@EnableTransactionManagement(proxyTargetClass = true))
         * - 클래스 기반 프록시
         * - 기본값 (matchIfMissing = true)
         */
        @Configuration(proxyBeanMethods = false)
        @EnableTransactionManagement(proxyTargetClass = true)
        @ConditionalOnProperty(prefix = &quot;spring.aop&quot;, name = &quot;proxy-target-class&quot;, havingValue = &quot;true&quot;,
                matchIfMissing = true)
        public static class CglibAutoProxyConfiguration {
            // 설정용 빈만 존재, 실제 로직은 없음
        }

    }

    /**
     * AspectJ 트랜잭션 처리 지원
     * - @Transactional을 AspectJ 방식으로 사용하는 경우
     * - AbstractTransactionAspect 빈이 존재해야 적용됨
     */
    @Configuration(proxyBeanMethods = false)
    @ConditionalOnBean(AbstractTransactionAspect.class)
    static class AspectJTransactionManagementConfiguration {

        /**
         * AbstractTransactionAspect는 지연 초기화 대상에서 제외해야 함 (즉시 초기화 필요)
         */
        @Bean
        static LazyInitializationExcludeFilter eagerTransactionAspect() {
            return LazyInitializationExcludeFilter.forBeanTypes(AbstractTransactionAspect.class);
        }

    }

}
</code></pre>
<p><code>TransactionAutoConfiguration</code>은 Spring Boot에서 <code>@EnableTransactionManagement</code>를 자동으로 등록해주는 역할을 합니다.
즉, 스프링 부트 환경이 아니면 개발자가 직접 <code>@EnableTransactionManagement</code>를 붙여야 합니다.</p>
<p><strong>TransactionAutoConfiguration은 자동 등록의 트리거일 뿐</strong>
여기서 주의깊게 보아야할 부분은 <strong><code>TransactionTemplateConfiguration</code></strong> 과 <strong><code>EnableTransactionManagementConfiguration</code></strong> 입니다.</p>
<h3 id="transactiontemplateconfiguration">TransactionTemplateConfiguration</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/739d070c-3c43-43f0-95c5-00051fb27437/image.png" /></p>
<p><code>TransactionTemplate</code>은 Spring에서 명시적 프로그래밍 방식으로 트랜잭션을 관리할 수 있게 해주는 템플릿 클래스입니다. 보통 <code>@Transactional</code>은 선언적 방식인데, 이건 프로그래밍 방식으로 트랜잭션을 제어하고자 할 때 사용합니다.</p>
<pre><code class="language-java">// 사용 예시
TransactionTemplate txTemplate = new TransactionTemplate(transactionManager);
String result = txTemplate.execute(status -&gt; {
    // 트랜잭션 안에서 실행할 코드
    someRepository.save(...);
    return &quot;성공&quot;;
});
</code></pre>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/abed530f-af90-4977-a5c3-8f6a6806a822/image.png" /> </p>
<blockquote>
<p>초기에는 트랜잭션을 사용하기 위해 <code>PlatformTransactionManager</code>를 직접 호출해 트랜잭션을 시작하고, 성공 시 커밋하거나 예외 발생 시 롤백하는 코드를 작성해야 했습니다. 이로 인해 동일한 트랜잭션 제어 로직이 여러 클래스에서 반복적으로 사용되는 문제가 발생했습니다.</p>
</blockquote>
<p>이를 해결하기 위해 <strong><code>TransactionTemplate</code></strong>이 도입되었습니다. <code>TransactionTemplate</code>을 사용하면 템플릿 콜백 패턴을 통해 트랜잭션 처리 코드를 일관되게 작성할 수 있고, 트랜잭션 시작, 커밋, 롤백 로직을 템플릿이 대신 처리해줍니다. 이 덕분에 반복되는 트랜잭션 제어 코드를 제거할 수 있게 되었습니다.</p>
<p>하지만 여전히 문제는 남아 있었습니다. <code>TransactionTemplate</code>을 사용하는 방식에서도 <strong>비즈니스 로직과 트랜잭션 처리 로직</strong>이 같은 클래스 안에 섞여 있어 두 관심사를 하나의 클래스에서 처리하게 되므로 관심사 분리가 어렵고 유지보수가 불편하다는 점입니다.</p>
<p>이 문제를 해결하기 위해 스프링은 AOP 기반의 선언적 트랜잭션 처리 방식을 제공합니다. <code>@Transactional</code> 어노테이션을 사용하면, 트랜잭션 경계 설정을 <strong>AOP 프록시가 대신 처리</strong>해주기 때문에, 개발자는 오직 비즈니스 로직에만 집중할 수 있습니다. 트랜잭션의 시작과 종료, 롤백 여부 등은 AOP 프레임워크가 자동으로 처리합니다.</p>
<h4 id="q-만약-transactional을-사용한다면-transactiontemplate을-사용하지-않는거네">Q. 만약 @Transactional을 사용한다면 TransactionTemplate을 사용하지 않는거네?</h4>
<p>맞습니다. <code>@Transactional</code> 을 사용하게 된다면 AOP 기반의 트랜잭션 처리를 이용하는 것으로  <code>TransactionTemplate</code>은 전혀 등장하지 않습니다.
대신 <strong><code>TransactionInterceptor</code></strong> 가 메서드 호출을 가로채서 기존에 <code>TransactionTemplate</code>이 수행했던 트랜잭션 시작, 커밋, 롤백 로직을 대신 수행해주게 됩니다.</p>
<h3 id="enabletransactionmanagementconfiguration">EnableTransactionManagementConfiguration</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/4c36bb9f-a69c-4ae1-a22f-9abb8ab44d5d/image.png" /></p>
<p>해당 클래스는 Spring Boot에서 <code>@EnableTransactionManagement</code>를 자동 설정하는 <strong>자동 구성 클래스</strong>입니다. 구체적으로는, AOP 기반 트랜잭션 처리에서 JDK 동적 프록시(JDK Proxy)와 CGLIB 프록시 중 어떤 것을 사용할지 자동으로 설정해주는 역할을 합니다.</p>
<h4 id="enabletransactionmanagement">@EnableTransactionManagement...?</h4>
<p><code>@EnableTransactionManagement</code>는 Spring에서 <code>@Transactional</code>이 실제로 동작하도록 활성화해주는 어노테이션입니다. -&gt; <strong>트랜잭션 AOP 설정을 활성화하는 역할 (환경 세팅)</strong></p>
<ul>
<li><code>@EnableTransactionManagement</code> 없으면 <code>@Transactional</code>은 무시됨 (프록시가 안 만들어짐)</li>
<li><code>@EnableTransactionManagement</code>만 있고 <code>@Transactional</code>이 없으면? → 아무 효과 없음</li>
</ul>
<p><strong>둘 다 있어야 AOP 기반 트랜잭션 처리가 정상적으로 작동</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/5900ac0f-5b06-4bdd-96aa-dd1889f4781c/image.png" />이 어노테이션이 적용되면 Spring은 <code>TransactionManagementConfigurationSelector</code>를 통해 다음 <strong>두 Bean을 등록합니다</strong></p>
<ul>
<li><code>AutoProxyRegistrar</code>: AOP 프록시 인프라 구성</li>
<li><code>ProxyTransactionManagementConfiguration</code>: <strong>Transaction Advisor와 Interceptor 등록</strong></li>
</ul>
<h4 id="autoproxyregistrar---빈-후처리기를-컨테이너에-등록하는-역할">AutoProxyRegistrar...? -&gt; <strong>빈 후처리기를 컨테이너에 등록하는 역할</strong></h4>
<p>이 클래스는 <code>InfrastructureAdvisorAutoProxyCreator</code>를 빈으로 등록합니다. 이 빈(<code>InfrastructureAdvisorAutoProxyCreator</code>)은 <code>@Transactional</code>이 붙은 메서드를 자동으로 프록시로 감싸줍니다 (즉, 동작을 가로채기 위한 프록시 생성)
즉, <code>AutoProxyRegistrar</code> 클래스는 <code>@EnableTransactionManagement</code>, <code>@EnableAsync</code>, <code>@EnableAspectJAutoProxy</code> 등
<code>AOP</code> 기반 기능을 활성화하는 어노테이션들에 의해 등록되는 클래스입니다.</p>
<p>이 클래스의 역할은 <code>AOP</code> 기능을 수행할 <code>AutoProxyCreator</code> 빈을 등록하는 것입니다. 
즉, 실제 AOP를 수행하는 객체는 아니고, 프록시를 생성해줄 객체(<strong>AutoProxyCreator</strong>)를 스프링 컨테이너에 등록해주는 역할을 합니다. </p>
<p><strong>내부에서는..</strong></p>
<ul>
<li><code>@EnableTransactionManagement</code> → ImportSelector로 <code>AutoProxyRegistrar</code>를 import</li>
<li><code>AutoProxyRegistrar.registerBeanDefinitions()</code>에서 <code>InfrastructureAdvisorAutoProxyCreator(=AutoProxyCreator)</code> 같은 프록시 생성기 빈을 BeanDefinitionRegistry에 등록 (아래 사진 참고)</li>
</ul>
<p>이후 Spring이 빈을 생성할 때 이 <code>AutoProxyCreator</code>가 개입해서 프록시 객체(AOP 대상)를 감싸도록 합니다.</p>
<p><strong>그럼 진짜 프록시를 만드는 주체는?</strong></p>
<ul>
<li><code>AutoProxyRegistrar</code>가 아니라 <code>AutoProxyCreator</code>들이 실제 프록시를 생성합니다.</li>
</ul>
<p><strong>대표적인 AutoProxyCreator</strong></p>
<ul>
<li><code>InfrastructureAdvisorAutoProxyCreator</code> : <strong>@Transactional, @Async 등 인프라 수준의 AOP용</strong></li>
<li><code>AnnotationAwareAspectJAutoProxyCreator</code> : <strong>@Aspect 기반 AOP용</strong></li>
<li><code>BeanNameAutoProxyCreator</code> : <strong>빈 이름으로 지정된 AOP 대상 프록시 생성</strong></li>
</ul>
<blockquote>
<p><code>AnnotationAwareAspectJAutoProxyCreator</code>는 @Aspect 기반 AOP를 처리하기 위한 <strong>빈 후처리기 (BeanPostProcessor)</strong>.
그리고 이건 우리가 흔히 말하는 커스텀 AOP (예: 로깅, 인증 체크, 성능 측정 등) 에서 동작하는 핵심 컴포넌트.</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/2915c712-4843-4ea5-9d14-6e06b661cc8b/image.png" /></p>
<p>결국은 <code>AutoProxyRegistrar</code>을 통해 <strong>AOP 프록시 생성기</strong>가 생성된다는 것을 알았습니다. 
정리하자면 </p>
<ul>
<li><code>AutoProxyRegistrar</code> 클래스는 <code>InfrastructureAdvisorAutoProxyCreator</code>를 빈으로 등록 </li>
<li><code>InfrastructureAdvisorAutoProxyCreator</code> 은 <code>@Transactional</code>이 붙은 메서드를 자동으로 프록시로 감싸줌</li>
</ul>
<h4 id="proxytransactionmanagementconfiguration">ProxyTransactionManagementConfiguration...?</h4>
<p><code>ProxyTransactionManagementConfiguration</code>는 트랜잭션을 위한 <code>Advisor</code> 및 <code>Interceptor</code> 등록하는 역할을 수행합니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/a7cbfe9d-2193-4836-9cbf-297a1ef247bf/image.png" /></p>
<ul>
<li><code>TransactionInterceptor</code> (트랜잭션 로직을 수행)</li>
<li><code>TransactionAttributeSourceAdvisor</code> (Advisor이며 Pointcut + Advice 형태)</li>
<li><em>이 Advisor는 @Transactional 어노테이션을 인식*</em></li>
</ul>
<p>최종적으로 <code>@EnableTransactionManagement</code>에 의해 등록된 <code>AutoProxyRegistrar</code>와 <code>ProxyTransactionManagementConfiguration</code>을 통해 <code>@Transactional</code>이 붙은 메서드는 프록시로 감싸지게 됩니다.
이 프록시는 내부적으로 <code>TransactionInterceptor</code>를 호출합니다.</p>
<h3 id="transactionaspectsupport-transactioninterceptor의-부모-클래스">TransactionAspectSupport (=TransactionInterceptor의 부모 클래스)</h3>
<p><code>TransactionInterceptor</code>는 상속받은 <code>TransactionAspectSupport</code>의 <code>invokeWithinTransaction()</code> 메서드를 통해 트랜잭션을 시작하고, 커밋하거나 롤백하는 과정을 수행합니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/24e67ef1-b002-43ea-a1fd-ba86e34941a5/image.png" /></p>
<p>큰 흐름을 정리하면 아래와 같습니다.</p>
<h3 id="스프링-선언적-트랜잭션transactional의-큰-흐름">스프링 선언적 트랜잭션(<code>@Transactional</code>)의 큰 흐름</h3>
<pre><code class="language-java">
@EnableTransactionManagement (자동 적용됨)
    ↓
TransactionManagementConfigurationSelector
    ↓
 ┌────────────────────────┬────────────────────────────┐
 │ AutoProxyRegistrar     │ ProxyTransactionConfig     │
 │ → AOP Creator 등록     │ → TransactionInterceptor   │
 │                        │ → TransactionAdvisor       │
 └────────────────────────┴────────────────────────────┘
    ↓
@MyService 등록됨 → 이때 @Transactional 프록시로 감쌈
    ↓
MyService.method() 호출 → 프록시의 invoke()
    ↓
TransactionInterceptor → invokeWithinTransaction()
    ↓
PlatformTransactionManager → getTransaction() / commit() / rollback()

</code></pre>
<p>이렇게 해서 <code>TransactionAutoConfiguration</code> 부터 시작해서 <code>TransactionInterceptor</code>까지의 과정을 통해 스프링 내부에서 선언적 트랜잭션이 적용되는 전반적인 과정을 알 수 있었습니다. 다음에는 <code>TransactionManager</code>의 동작 방식에 대해 알아보겠습니다.</p>
<blockquote>
<p>아래는 <a href="https://velog.io/@ho-tea/%EC%9E%90%EB%8F%99%EA%B5%AC%EC%84%B1..%EA%B7%BC%EB%8D%B0-%EC%9D%B4%EC%A0%9C-Data-JPA%EB%A5%BC-%EA%B3%81%EB%93%A4%EC%9D%B8">자동구성..근데-이제-Data-JPA를-곁들인</a> 부터 시작해서 지금까지의 흐름을 정리한 표입니다.</p>
</blockquote>
<h3 id="🔄-spring-boot-컨테이너-라이프사이클--transactional-적용-흐름-정리">🔄 Spring Boot 컨테이너 라이프사이클 + @Transactional 적용 흐름 정리</h3>
<table>
<thead>
<tr>
<th>단계</th>
<th>설명</th>
<th>관련 클래스 또는 어노테이션</th>
<th>생명주기 타이밍</th>
</tr>
</thead>
<tbody><tr>
<td>0️⃣</td>
<td>애플리케이션 시작, <code>SpringApplication.run()</code> 호출</td>
<td><code>SpringApplication</code></td>
<td>애플리케이션 부트</td>
</tr>
<tr>
<td>1️⃣</td>
<td>사용자 정의 <code>@Component</code>, <code>@Service</code> 등의 스캔 시작</td>
<td><code>@ComponentScan</code>, <code>ConfigurationClassPostProcessor</code></td>
<td>빈 정의 등록 시작 전</td>
</tr>
<tr>
<td>2️⃣</td>
<td>사용자 정의 빈 구성 클래스(<code>@Configuration</code> 등) 먼저 처리됨</td>
<td><code>@Configuration</code>, <code>@Bean</code></td>
<td>등록 우선순위 ↑</td>
</tr>
<tr>
<td>3️⃣</td>
<td><code>@EnableAutoConfiguration</code>에 따라 AutoConfig 클래스 로딩</td>
<td><code>spring.factories</code> → <code>META-INF</code></td>
<td>자동 구성 클래스 탐색 시점</td>
</tr>
<tr>
<td>4️⃣</td>
<td><code>DataSourceAutoConfiguration</code> 실행 → HikariDataSource 등록</td>
<td><code>DataSourceAutoConfiguration</code></td>
<td>DB 연결 구성</td>
</tr>
<tr>
<td>5️⃣</td>
<td><code>HibernateJpaAutoConfiguration</code> 실행</td>
<td><code>JpaVendorAdapter</code>, <code>EntityManagerFactoryBuilder</code></td>
<td>DataSource 이후</td>
</tr>
<tr>
<td>→</td>
<td><code>HibernateJpaConfiguration</code> 로드</td>
<td><code>LocalContainerEntityManagerFactoryBean</code></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>JpaTransactionManager</code>, <code>EntityManagerFactory</code> 등록</td>
<td><code>JpaBaseConfiguration</code> 상속</td>
<td></td>
</tr>
<tr>
<td>6️⃣</td>
<td><code>JpaRepositoriesAutoConfiguration</code> 실행</td>
<td>Repository 스캔</td>
<td>Hibernate 이후</td>
</tr>
<tr>
<td>7️⃣</td>
<td><code>TransactionAutoConfiguration</code> 실행</td>
<td>트랜잭션 관련 설정</td>
<td>DataSource, JPA 이후</td>
</tr>
<tr>
<td>→</td>
<td><code>EnableTransactionManagementConfiguration</code> 내부에서</td>
<td></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>@EnableTransactionManagement</code> 자동 적용</td>
<td><strong>중요 트리거</strong></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>TransactionManagementConfigurationSelector</code> → Import 두 개</td>
<td></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>AutoProxyRegistrar</code> → <code>InfrastructureAdvisorAutoProxyCreator</code> 등록</td>
<td>AOP 프록시 생성기</td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>ProxyTransactionManagementConfiguration</code> → <code>TransactionInterceptor</code>, <code>TransactionAdvisor</code> 등록</td>
<td>어드바이저 구성 완료</td>
<td></td>
</tr>
<tr>
<td>8️⃣</td>
<td>이 시점에 <code>@Service</code>, <code>@Component</code> 등 사용자 정의 빈 등록 시작</td>
<td>일반적인 빈 생성 시점</td>
<td></td>
</tr>
<tr>
<td>9️⃣</td>
<td>등록된 <code>BeanPostProcessor</code> 동작 시작</td>
<td><code>InfrastructureAdvisorAutoProxyCreator</code> 등</td>
<td></td>
</tr>
<tr>
<td>→</td>
<td>Advisor 조건 만족 시 프록시 감싸짐 (<code>@Transactional</code> 등)</td>
<td>✅ <strong>프록시 객체 생성</strong></td>
<td></td>
</tr>
<tr>
<td>🔟</td>
<td>사용자 코드에서 메서드 호출 시 → 프록시 객체가 가로채기</td>
<td></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>TransactionInterceptor.invoke()</code> 실행됨</td>
<td></td>
<td></td>
</tr>
<tr>
<td>→</td>
<td>내부적으로 <code>invokeWithinTransaction()</code> 호출</td>
<td>트랜잭션 시작, 커밋, 롤백 처리</td>
<td></td>
</tr>
<tr>
<td>→</td>
<td><code>PlatformTransactionManager.getTransaction()</code> → <code>commit()</code> or <code>rollback()</code></td>
<td><strong>실질 트랜잭션 처리</strong></td>
<td></td>
</tr>
</tbody></table>
<h3 id="🔄-spring-bean-lifecycle--transactional-적용-지점-정리">🔄 Spring Bean Lifecycle + @Transactional 적용 지점 정리</h3>
<table>
<thead>
<tr>
<th>라이프사이클 단계</th>
<th>설명</th>
<th>기존 흐름에서 해당하는 단계</th>
</tr>
</thead>
<tbody><tr>
<td>1️⃣ 빈 정의 등록 (Definition 등록)</td>
<td>어떤 빈이 존재할 것인지 스프링이 &quot;정의&quot;만 먼저 등록하는 단계</td>
<td>① ~ ⑦ 전체<br />- <code>@ComponentScan</code><br />- <code>@EnableAutoConfiguration</code><br />- <code>AutoProxyRegistrar</code>, <code>TransactionInterceptor</code> 등도 이 시점 등록</td>
</tr>
<tr>
<td>2️⃣ 빈 인스턴스 생성</td>
<td>정의된 빈을 바탕으로 객체를 <code>new</code>해서 인스턴스를 만드는 단계</td>
<td>⑧ 사용자 정의 빈 생성 시점<br />예: <code>@Service</code>, <code>@Component</code> 클래스</td>
</tr>
<tr>
<td>3️⃣ 의존성 주입</td>
<td>생성된 빈에 필요한 의존 객체를 <code>@Autowired</code>, 생성자 등으로 주입</td>
<td>⑧과 함께 진행</td>
</tr>
<tr>
<td>4️⃣ 초기화 (PostProcessor 포함)</td>
<td><code>InitializingBean.afterPropertiesSet()</code>, <code>@PostConstruct</code><br />+ <code>BeanPostProcessor</code> 적용<br />→ 이 시점에 프록시 감싸짐</td>
<td>⑨<br />- <code>InfrastructureAdvisorAutoProxyCreator</code> 작동<br />- <code>@Transactional</code> 조건 만족 시 프록시로 감쌈</td>
</tr>
<tr>
<td>5️⃣ 사용</td>
<td>사용자가 메서드 호출 → 프록시가 가로채서 트랜잭션 처리 시작</td>
<td>🔟 이후<br />- <code>TransactionInterceptor.invoke()</code><br />- <code>invokeWithinTransaction()</code> 내부에서 트랜잭션 시작, 커밋, 롤백</td>
</tr>
<tr>
<td>6️⃣ 소멸</td>
<td>빈이 컨테이너에서 제거될 때 실행<br /><code>@PreDestroy</code>, <code>DisposableBean.destroy()</code> 등</td>
<td>트랜잭션과는 직접적인 관련 없음</td>
</tr>
</tbody></table>
<ul>
<li><code>@Transactional</code>의 실제 적용 타이밍은 빈 초기화 직전인 <code>BeanPostProcessor</code> 단계에서 발생.</li>
<li>즉, 빈이 생성된 후에 프록시로 감싸질 수 있는지 조건 판단 → 감싸기가 이뤄짐.</li>
<li>감싸진 후, 메서드가 호출될 때 트랜잭션이 동작하고, 이건 Bean 사용 단계에 해당.</li>
</ul>