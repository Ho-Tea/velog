<blockquote>
<h3 id="오늘의-목표">오늘의 목표</h3>
<p><code>DataSource</code>가 2개 이상일때 자동구성 과정 알아보기</p>
</blockquote>
<p>일전에 <a href="https://velog.io/@ho-tea/%EC%9E%90%EB%8F%99%EA%B5%AC%EC%84%B1..%EA%B7%BC%EB%8D%B0-%EC%9D%B4%EC%A0%9C-Data-JPA%EB%A5%BC-%EA%B3%81%EB%93%A4%EC%9D%B8">Data JPA를 사용한 환경에서의 자동구성 과정</a> 를 통해 자동 구성에서 호출되는 클래스와 등록되는 빈들을 살펴보았습니다. 실제 프로젝트에서는 단일 데이터베이스 대신 리플리케이션 등 다중화 작업을 진행하는 경우가 많다고 생각합니다. 그래서 이번에는 하나의 <code>DataSource</code>가 아닌, 여러 <code>DataSource</code>가 등록된 환경에서 자동 구성이 어떻게 이루어지는지 알아보겠습니다. <strong>(짧음 주의)</strong></p>
<h3 id="리플리케이션-환경-구성">리플리케이션 환경 구성</h3>
<p><strong>yml 파일</strong></p>
<p>우선 여러개의 <code>DataSource</code>를 등록해놓아야 하는 상태이므로 아래와 같이 <code>yml</code> 파일을 설정해 주었습니다. 
(간단하게 <code>Reader DB</code>와 <code>Writer DB</code>로 구성해놓았습니다.)</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/bda8e0db-87d5-4c1b-af27-5aa65847e36c/image.png" /></p>
<p><strong>production code</strong></p>
<ul>
<li>여러 <code>DataSource</code>(읽기와 쓰기)를 동적으로 라우팅하고, <code>LazyConnectionDataSourceProxy</code>를 통해 실제 연결 시점을 늦춤으로써, 트랜잭션이나 데이터베이스 작업이 실제로 필요할 때까지 연결을 열지 않도록 합니다.</li>
<li>이 방식은 런타임에 어떤 데이터 소스를 사용할지 결정할 수 있게 해주며, 예를 들어 쓰기 작업은 <code>writer</code>, 읽기 작업은 <code>reader</code>로 분리할 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/600cec6e-ef36-4e68-b4df-29ff63fe20ce/image.png" /></li>
</ul>
<p>이와 같이 설정한 후 테스트를 수행해보면 정상적으로 테스트가 통과되는 것을 알 수 있습니다.
또한, 아래 콘솔에 출력된 <code>Bean</code>을 확인해보니 <code>LazyConnectionDataSourceProxy</code> 가 반환된 것을 알 수 있습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/8fc99a6a-d45f-4760-963c-57f5c71987c2/image.png" /></p>
<h3 id="datasource-bean을-따로-생성하지-않는다면">DataSource Bean을 따로 생성하지 않는다면?</h3>
<p><code>production code</code>를 모두 지우고 다시 테스트를 수행하면 <code>DataSource</code>를 특정지을 수 없다는 에러 메세지가 나오게 되는데요. 
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/4ed1b1fc-475d-4f97-82f1-b7fc22446ba8/image.png" /></p>
<blockquote>
<p>Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'dataSourceScriptDatabaseInitializer' defined in class path resource [org/springframework/boot/autoconfigure/sql/init/DataSourceInitializationConfiguration.class]: Unsatisfied dependency expressed through method 'dataSourceScriptDatabaseInitializer' parameter 0: Error creating bean with name 'dataSource' defined in class path resource [org/springframework/boot/autoconfigure/jdbc/DataSourceConfiguration$Hikari.class]: Failed to instantiate [com.zaxxer.hikari.HikariDataSource]: Factory method 'dataSource' threw exception with message: Failed to determine a suitable driver class</p>
</blockquote>
<p>단일 <code>DataSource</code> 설정이 있으면 <code>Spring Boot</code>의 자동 구성에 의해 기본 <code>DataSource</code> 빈이 생성되지만, 두 개 이상의 <code>DataSource</code>가 존재하면 어떤 <code>DataSource</code>를 기본으로 사용할지 결정할 수 없어서 &quot;DataSource를 특정지을 수 없습니다&quot;라는 에러가 발생하였습니다. 
이를 해결하려면, 여러 <code>DataSource</code> 중 하나(아까의 <code>LazyConnectionDataSourceProxy</code>)를 <code>@Primary</code>로 지정하거나, 명시적으로 기본 <code>DataSource</code>를 선택할 수 있는 추가 구성을 해야 합니다.</p>
<p>예외 메세지를 더 살펴보자면,</p>
<ul>
<li>빨간색 박스를 자세히 보면 <code>dataSourceInitializationConfiguration</code>의 <code>dataSourceScriptDatabaseInitializer()</code> 메서드의 첫번재 인자로 <code>DataSource</code>가 들어와야 하는데 해당 <code>DataSource</code>가 정상적으로 생성이 되지 않았다는 예외 메세지가 존재합니다.</li>
<li>예외를 따라가다보면 <code>DataSourceConfiguration</code> 클래스가 보이게 됩니다. <a href="https://velog.io/@ho-tea/%EC%9E%90%EB%8F%99%EA%B5%AC%EC%84%B1..%EA%B7%BC%EB%8D%B0-%EC%9D%B4%EC%A0%9C-Data-JPA%EB%A5%BC-%EA%B3%81%EB%93%A4%EC%9D%B8">Data JPA를 사용한 환경에서의 자동구성 과정</a> 에서도 다뤘다시피 <code>DataSourceConfiguration</code>는 <code>DataSource</code>를 생성하는 곳이지만 생성이 이루어지지 않았습니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/56dc6f7d-b617-4e0f-a030-fe3319778da5/image.png" /></li>
<li>애초에 <code>DataSourceProperties</code> 에 내부 값들이 <code>null</code>로 입력된 것을 알 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/29075d36-37f8-4cf9-8284-46faee23f8f0/image.png" /><blockquote>
<p><code>@ConfigurationProperties(prefix = &quot;spring.datasource&quot;)</code> 어노테이션은 Spring Boot에서 외부 설정 파일(예: application.yml 또는 application.properties)에 정의된 <code>spring.datasource</code>로 시작하는 속성들을 Java 객체의 필드에 바인딩하기 위해 사용되지만,
현재는 <code>DataSource</code>를 2개 정의하는 바람에 해당 속성을 사용하지 못하였기에 자동으로 <code>DataSource</code>를 설정할 수 없었습니다.</p>
</blockquote>
</li>
</ul>
<h3 id="datasource를-2개-이상-바인딩-할-수-있게-스프링이-지원해줄수는-없나">DataSource를 2개 이상 바인딩 할 수 있게 스프링이 지원해줄수는 없나?</h3>
<p>해당 글을 쓰면서 가장 먼저 든 생각이였습니다..ㅎ
이러한 생각을 이미 스프링은 예전부터 해왔지만.. 아직 해결되지는 않은것으로 보입니다.
 -&gt; <a href="https://github.com/spring-projects/spring-boot/issues/15732">스프링 이슈</a></p>
<h3 id="어떻게-해결하지">어떻게 해결하지?</h3>
<p>제가 선택한 해결 방법은 단순합니다. 
기존에 자동구성으로 <code>DataSource</code>가 생성되고, <code>EntityManagerFactory</code>가 생성되고, <code>JpaTransactionManager</code>가 생성이 되었기에, <code>DataSource</code>만 따로 정의한 빈이 등록되게 하고, 나머지 <code>JpaTransactionManager</code>와 <code>EntityManagerFactory</code> 등 <code>JPA</code> 관련 빈들은 <code>Spring Boot</code>가 생성되게 구성하는 것으로 해결하고자 하였습니다. 따라서 여러 <code>DataSource</code>를 빈으로 생성한 후 <code>@Primary</code>를 통해 빈에 우선권을 주었습니다.</p>
<blockquote>
<p>개발자가 정의한 <code>Component</code>들이 먼저 스캔 돼서 <code>Bean</code>으로 등록된 후에, <code>AutoConfiguration</code>의 <code>Bean</code>들이 등록되기 때문에 <code>@ConditionalOnSingleCandidate(DataSource.class)</code> 을 사용하는 (예시:<code>HibernateJpaConfiguration</code>) 클래스들의 조건만 만족시켜주면 스프링 부트의 자동구성을 이용할 수 있습니다.</p>
</blockquote>