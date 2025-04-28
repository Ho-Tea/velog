<blockquote>
<h3 id="오늘의-목표">오늘의 목표</h3>
<p>스프링에서의 트랜잭션 추상화와 동작 순서 그리고 트랜잭션 매니저 종류에 대해 알아보겠습니다.</p>
</blockquote>
<p><a href="https://velog.io/@ho-tea/TransactionAutoConfiguration-%EA%B7%B8-%ED%9B%84%EB%A1%9C">TransactionAutoConfiguration 그 후로...</a>를 통해 최종적으로 아래와 같은 <code>flow</code>로 <code>PlatformTransactionManager</code>이 트랜잭션을 시작한다는 것을 알 수 있었습니다.</p>
<pre><code class="language-java">... 
    ↓
MyService.method() 호출 → 프록시의 invoke()
    ↓
TransactionInterceptor → invokeWithinTransaction()
    ↓
PlatformTransactionManager → getTransaction() / commit() / rollback()
</code></pre>
<p><a href="https://velog.io/@ho-tea/%EC%9E%90%EB%8F%99%EA%B5%AC%EC%84%B1..%EA%B7%BC%EB%8D%B0-%EC%9D%B4%EC%A0%9C-Data-JPA%EB%A5%BC-%EA%B3%81%EB%93%A4%EC%9D%B8">자동구성..근데-이제-Data-JPA를-곁들인</a> 을 통해 <code>JPA</code>를 사용하게 되었을 때는 <code>JpaTransactionManager</code>가 등록된다는 것을 알 수 있었는데요. 
마지막에 <code>JpaTransactionManager</code>가 아닌 <code>PlatformTransactionManager</code> 을 통해 <code>transaction</code>을 실행한다는 것이 조금은 어색하게 다가옵니다.</p>
<blockquote>
<p><strong>JpaTransactionManager</strong></p>
</blockquote>
<ul>
<li><code>JPA</code> 기반의 <code>DataSource</code>를 사용하는 경우에 트랜잭션을 관리해주는 스프링의 트랜잭션 매니저 구현체
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/eacecb83-519d-4032-8e33-05d9c53269e9/image.png" /></li>
</ul>
<blockquote>
<p><strong>DataSourceTransactionManager</strong></p>
</blockquote>
<ul>
<li><code>JDBC</code> 기반의 <code>DataSource</code>를 사용하는 경우에 트랜잭션을 관리해주는 스프링의 트랜잭션 매니저 구현체
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/da0a5e10-9d4d-4aa4-801a-a00b159530ce/image.png" /></li>
</ul>
<blockquote>
<p><strong>AbstractPlatformTransactionManager</strong>
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/b2efaf4a-63cf-40a6-9617-748a29dfdda6/image.png" /></p>
</blockquote>
<p><code>JpaTransactionManager</code> 코드를 살펴보면 그 이유를 알 수 있는데요. <code>JpaTransactionManager</code>은 <code>AbstractPlatformTransactionManager</code>을 상속하고 <code>AbstractPlatformTransactionManager</code>는 <code>PlatformTransactionManager</code>를 확장한 형태인 것을 확인할 수 있습니다.</p>
<p><strong>이를 통해 <code>PlatformTransactionManager</code>는 <code>JpaTransactionManager</code>을 추상화한 인터페이스임을 알 수 있습니다.</strong></p>
<p>이번 글에서는 스프링이 제공하는 이러한 <strong>트랜잭션 추상화 기능</strong>과 <strong>리소스 동기화</strong>에 대해 살펴보겠습니다.</p>
<ul>
<li><strong>트랜잭션 추상화</strong></li>
<li><strong>리소스 동기화</strong> -&gt; 다음글에서 다룰 예정</li>
</ul>
<h2 id="트랜잭션-추상화">트랜잭션 추상화</h2>
<blockquote>
<p>트랜잭션을 시작하고, 커밋하거나 롤백하는 트랜잭션 제어 책임 객체 -&gt; <code>Transaction Manager</code> (=<code>DataSourceTransactionManager</code>, <code>JpaTransactionManager</code> 등)</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/4487f97d-922c-4ec5-bc3e-5776558af9f3/image.png" /></p>
<p>구글에 <code>TransactionManager</code>를 검색하게 된다면 관련 검색어로 <code>PlatformTransactionManager</code>이 먼저 나오는 것을 알 수 있습니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/cc4723cc-0db4-4dad-b034-b968fe91b1db/image.png" /></p>
<h3 id="platformtransactionmanager">PlatformTransactionManager</h3>
<p> 스프링은 애플리케이션에서 트랜잭션을 보다 쉽게 관리할 수 있도록 하기 위해, 다양한 데이터 접근 기술(JDBC, JPA 등)에 종속되지 않는 트랜잭션 추상화를 제공합니다. 이 추상화의 핵심 인터페이스가 바로 <code>PlatformTransactionManager</code>입니다.
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/a9fd8e8b-1e37-40cb-9dd6-a709b26a0d2c/image.png" /></p>
<h3 id="왜-트랜잭션-추상화를-할까">왜 트랜잭션 추상화를 할까?</h3>
<p>초기에는 <code>JDBC</code>를 사용해 직접 트랜잭션을 제어했다가 나중에 <code>JPA</code>를 도입하는 경우, 서비스 계층의 코드가 데이터 접근 기술에 맞춰 수정되어야 하는 문제를 겪게 됩니다. 하지만 스프링의 <code>PlatformTransactionManager</code>를 사용하면, 해당 기술별 구현체(<code>DataSourceTransactionManager</code> or <code>JpaTransactionManager</code> 등)를 선택하여 적용할 수 있으므로, 서비스 계층에서는 동일한 트랜잭션 관리 코드를 사용할 수 있습니다. 
<strong>즉, 데이터 접근 기술이 변경되더라도 애플리케이션 코드의 변경 없이 트랜잭션 관리를 유지할 수 있게 됩니다.</strong></p>
<blockquote>
<p><strong>JpaTransactionManager 등록</strong>  - <code>JPA</code> <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/36c05e64-af98-48d1-8a3d-c76998642388/image.png" /> <strong>DataSourceTransactionManager 등록</strong> - <code>JDBC</code>
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/c4349e81-a07c-485c-896b-3351700051c1/image.png" /><code>transactionManager()</code> 메서드에 동일하게 <code>@ConditionalOnMissingBean(TransactionManager.class)</code>가 존재합니다.
이건 <code>PlatformTransactionManager</code> 타입 빈이 존재하지 않을 때만 등록하라는 뜻으로 내부적으로 <code>TransactionManager.class = PlatformTransactionManager.class</code> 라고 생각해도 무방합니다.</p>
</blockquote>
<h3 id="내-프로젝트에는-어떤-transactionmanager가-일을하고-있을까">내 프로젝트에는 어떤 TransactionManager가 일을하고 있을까?</h3>
<h4 id="data-jdbc일-경우---jdbctransactionmanager">Data-JDBC일 경우 -&gt; JdbcTransactionManager</h4>
<ul>
<li><code>implementation 'org.springframework.boot:spring-boot-starter-data-jdbc'</code>
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/8ffc2547-a104-4a72-9171-7f4daa98d552/image.png" />
현재 <code>JdbcTransactionManager</code>가 <code>TransactionManager</code>의 구현체로 등록이 되어있는데 <code>JdbcTransactionManager</code>는 <code>Spring 6.0</code> 부터 <code>DataSourceTransactionManager</code>를 확장하여 거의 모든 기능을 그대로 사용하면서 이름을 더 직관적으로 바꾼 것이라고 보면 됩니다.<blockquote>
<ul>
<li><code>JdbcTransactionManager</code>
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/0bc472d6-e680-4532-b378-69204290aff0/image.png" /></li>
</ul>
</blockquote>
</li>
</ul>
<h4 id="data-jpa일-경우---jpatransactionmanager">Data-JPA일 경우 -&gt; JpaTransactionManager</h4>
<ul>
<li><code>implementation 'org.springframework.boot:spring-boot-starter-data-jpa'</code>
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/3f6fdc78-696a-4901-8510-b55e7ee02914/image.png" /></li>
</ul>
<h3 id="jdbc-vs-data-jdbc-vs-jpa-vs-data-jpa">JDBC vs Data JDBC vs JPA vs Data JPA</h3>
<table>
<thead>
<tr>
<th>구분</th>
<th>JDBC</th>
<th>Spring Data JDBC</th>
<th>JPA (with Hibernate)</th>
<th>Spring Data JPA</th>
</tr>
</thead>
<tbody><tr>
<td><strong>핵심 목표</strong></td>
<td>DB와 직접 통신</td>
<td>간단한 CRUD 자동화</td>
<td>ORM 기반 객체-DB 매핑</td>
<td>JPA 위에 추상화된 CRUD + 쿼리 자동화</td>
</tr>
<tr>
<td><strong>추상화 수준</strong></td>
<td>가장 낮음 (SQL 직접 작성)</td>
<td>중간 (도메인 매핑 + SQL 일부 자동화)</td>
<td>높은 수준의 ORM</td>
<td>ORM + 자동 Repository 구현</td>
</tr>
<tr>
<td><strong>사용 방식</strong></td>
<td><code>Connection</code>, <code>Statement</code>, <code>ResultSet</code> 또는 <code>JdbcTemplate</code> 사용</td>
<td><code>JdbcTemplate</code> 기반 + Spring Data 철학에 따라 Repository 자동 생성 <br />➡️ <em>Repository는 인터페이스로서, DAO(데이터 접근 객체) 역할을 대신함</em></td>
<td><code>EntityManager</code>를 사용해 엔티티 관리, 연관관계 매핑도 포함</td>
<td><code>JpaRepository</code> 상속만으로 CRUD 구현, <code>@Query</code>로 복잡 쿼리도 가능</td>
</tr>
<tr>
<td><strong>연관관계 매핑</strong></td>
<td>직접 <code>JOIN</code> SQL 작성</td>
<td>제한적 (1:1 정도만, 깊은 연관 매핑은 불가능)</td>
<td><code>@Entity</code>, <code>@OneToMany</code>, <code>@ManyToOne</code> 등을 통해 객체 간 연관관계 선언</td>
<td>JPA의 모든 매핑 기능 지원, 연관관계 탐색과 자동 조회 가능</td>
</tr>
<tr>
<td><strong>복잡 쿼리 처리</strong></td>
<td>SQL 직접 작성</td>
<td>SQL 기반 <code>@Query</code> 작성</td>
<td>JPQL (객체 지향 쿼리 언어), Criteria API</td>
<td>JPQL, <code>@Query</code>, QueryDSL, Specification 등 다양한 방식 지원</td>
</tr>
<tr>
<td><strong>학습 난이도</strong></td>
<td>낮음 (SQL 익숙하면 쉽게 시작)</td>
<td>중간 (Spring과 도메인 중심 모델 이해 필요)</td>
<td>높음 (ORM 개념, 연관관계, 영속성 컨텍스트 이해 필요)</td>
<td>중간 (Spring Data의 추상화 개념만 익히면 활용 쉬움)</td>
</tr>
<tr>
<td><strong>주요 구현체</strong></td>
<td>JDBC 표준 (Oracle, MySQL, H2 등 DB 드라이버)</td>
<td>Spring Data JDBC 모듈</td>
<td>Hibernate, EclipseLink, TopLink 등</td>
<td>Hibernate + Spring Data JPA 조합</td>
</tr>
</tbody></table>
<h3 id="트랜잭션-매니저-동작-방식">트랜잭션 매니저 동작 방식</h3>
<p>데이터 접근 기술이 변경되더라도 애플리케이션 코드의 변경 없이 트랜잭션 관리를 유지할 수 있게끔 <code>PlatformTransactionManager</code>이 사용된다는 것을 알 수 있었습니다. 이제는 <code>PlatformTransactionManager</code>의 구현체인 <code>JpaTrnasactionManager</code>을 사용하였을때 어떠한 방식으로 메서드 호출이 일어나는지 알아보겠습니다.</p>
<pre><code class="language-java">... 
    ↓
MyService.method() 호출 → 프록시의 invoke()
    ↓
TransactionInterceptor → invokeWithinTransaction()
    ↓
PlatformTransactionManager → getTransaction() / commit() / rollback()
</code></pre>
<ol>
<li><p>우선 <code>PlatformTransactionManager</code>의 추상메서드 <code>getTransaction()</code> 이 시작되면
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/3ea441ec-7fe7-490f-8bad-4cbe1b0a88c3/image.png" /></p>
</li>
<li><p>해당 메서드를 정의한 <code>AbstractPlatformTransactionManager</code>의 <code>getTransaction()</code>이 호출되게 됩니다. 
<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/3862cf79-c82f-4493-ab41-5153f47fd382/image.png" />
내부를 자세히 살펴보면 아래의 2가지 메서드 호출이 존재합니다.
```java</p>
</li>
<li><p>Object transaction = this.doGetTransaction();</p>
</li>
<li><p>return this.handleExistingTransaction(def, transaction, debugEnabled);</p>
<pre><code>#### `this.doGetTransaction();`
![](https://velog.velcdn.com/images/ho-tea/post/ec460e89-2de9-4b43-82ed-c21469c63f35/image.png)
``` java

 protected Object doGetTransaction() {
     JpaTransactionObject txObject = new JpaTransactionObject();
     txObject.setSavepointAllowed(this.isNestedTransactionAllowed());
     EntityManagerHolder emHolder = (EntityManagerHolder)TransactionSynchronizationManager.getResource(this.obtainEntityManagerFactory());
     if (emHolder != null) {
         if (this.logger.isDebugEnabled()) {
             this.logger.debug(&quot;Found thread-bound EntityManager [&quot; + emHolder.getEntityManager() + &quot;] for JPA transaction&quot;);
         }

         txObject.setEntityManagerHolder(emHolder, false);
     }

     if (this.getDataSource() != null) {
         ConnectionHolder conHolder = (ConnectionHolder)TransactionSynchronizationManager.getResource(this.getDataSource());
         txObject.setConnectionHolder(conHolder);
     }

     return txObject;
 }</code></pre></li>
</ol>
<ul>
<li><code>doGetTransaction()</code>은 추상화 메서드로 현재 쓰레드에서 이미 시작된 트랜잭션이 있는지 확인하고, 트랜잭션 객체(여기서는 <code>JpaTransactionObject</code>)를 생성해 반환합니다.</li>
<li>이 메서드는 트랜잭션의 시작 전 준비단계로서, 이미 쓰레드에 바인딩된 <code>EntityManager</code>나 <code>Connection</code>이 있으면 그것을 활용하고 없으면 이후 <code>doBegin()</code> 단계에서 새로 트랜잭션을 시작하도록 도와주는 역할을 합니다.</li>
</ul>
<h4 id="thishandleexistingtransactiondef-transaction-debugenabled"><code>this.handleExistingTransaction(def, transaction, debugEnabled);</code></h4>
<ul>
<li>이 메서드는 현재 쓰레드에 이미 존재하는 트랜잭션이 있을 때의 처리 흐름을 담당합니다. 즉, 전파 속성(<code>Propagation</code>)에 따라 트랜잭션을 어떻게 이어받거나 새로 만들지를 결정하는 중심 메서드입니다.</li>
<li>해당 메서드를 통해 <code>startTransaction()</code> 이 호출되는데, <code>handleExistingTransaction()</code> 자체는 조건에 따라 <code>startTransaction()</code>을 호출하지 않기도 합니다.</li>
<li>전파 속성에 따라 기존 트랜잭션에 참여하게 된다면 <code>prepareTransactionStatus()</code> 을 호출하게 됩니다.</li>
</ul>
<blockquote>
<p>여기서는 새롭게 트랜잭션을 만든다고 가정하고 진행하겠습니다. -&gt; <code>startTransaction()</code></p>
</blockquote>
<ol start="3">
<li><p><code>AbstractPlatformTransactionManager.handleExistingTransaction()</code> -&gt; <code>AbstractPlatformTransactionManager.startTransaction()</code> -&gt; <code>구현체.doBegin()</code></p>
<pre><code class="language-java"> private TransactionStatus startTransaction(TransactionDefinition definition, Object transaction, boolean nested, boolean debugEnabled, @Nullable SuspendedResourcesHolder suspendedResources) {
     boolean newSynchronization = this.getTransactionSynchronization() != 2;
     DefaultTransactionStatus status = this.newTransactionStatus(definition, transaction, true, newSynchronization, nested, debugEnabled, suspendedResources);
     this.transactionExecutionListeners.forEach((listener) -&gt; {
         listener.beforeBegin(status);
     });

     try {
         this.doBegin(transaction, definition);
     } catch (Error | RuntimeException var9) {
         Throwable ex = var9;
         this.transactionExecutionListeners.forEach((listener) -&gt; {
             listener.afterBegin(status, ex);
         });
         throw ex;
     }

     this.prepareSynchronization(status, definition);
     this.transactionExecutionListeners.forEach((listener) -&gt; {
         listener.afterBegin(status, (Throwable)null);
     });
     return status;
 }

</code></pre>
</li>
</ol>
<pre><code>protected void prepareSynchronization(DefaultTransactionStatus status, TransactionDefinition definition) {
    if (status.isNewSynchronization()) {
        TransactionSynchronizationManager.setActualTransactionActive(status.hasTransaction());
        TransactionSynchronizationManager.setCurrentTransactionIsolationLevel(definition.getIsolationLevel() != -1 ? definition.getIsolationLevel() : null);
        TransactionSynchronizationManager.setCurrentTransactionReadOnly(definition.isReadOnly());
        TransactionSynchronizationManager.setCurrentTransactionName(definition.getName());
        TransactionSynchronizationManager.initSynchronization();
    }

}

```</code></pre><p><code>startTransaction()</code> 에서 주의깊게 보아야 할 단계는 크게 3가지입니다. </p>
<table>
<thead>
<tr>
<th>단계</th>
<th>역할</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>1번째. 트랜잭션 상태 객체 생성</td>
<td>상태 저장</td>
<td><code>DefaultTransactionStatus</code> 객체를 생성하여, 현재 트랜잭션이 새 트랜잭션인지, 중첩 트랜잭션인지 등의 상태 정보를 저장합니다. 이 객체는 이후 커밋/롤백 판단 시 사용됩니다.</td>
</tr>
<tr>
<td>2번째. 실제 트랜잭션 시작</td>
<td>트랜잭션 열기</td>
<td><code>JpaTransactionManager#doBegin()</code> 메서드에서 <code>EntityManager.getTransaction().begin()</code> 을 호출하여 JPA 트랜잭션을 시작합니다. 이때 <code>EntityManager</code>는 트랜잭션 동기화 매니저에 바인딩됩니다.</td>
</tr>
<tr>
<td>3번째. 동기화 설정</td>
<td>컨텍스트 바인딩</td>
<td><code>TransactionSynchronizationManager</code>를 통해 현재 쓰레드에 트랜잭션 이름, 읽기 전용 여부, 격리 수준 등을 바인딩합니다. 이후 동작하는 DAO나 Repository는 이 정보를 참조하여 동일 트랜잭션에 참여할 수 있습니다.</td>
</tr>
</tbody></table>
<ul>
<li><p><code>1번째</code>에서 <code>DefaultTransactionStatus</code>는 트랜잭션의 현재 상태(신규 여부, 읽기 전용 여부, 저장점 보유 여부 등)를 표현하고 관리하는 객체입니다.<img alt="" src="https://velog.velcdn.com/images/ho-tea/post/fcf0f86d-1010-4c42-8db0-80fe8272f8a3/image.png" /></p>
</li>
<li><p><code>3번째</code>에서 <code>TransactionSynchronizationManager</code>를 통해 현재 쓰레드에 트랜잭션 <code>정보</code>를 바인딩하게 되는데, <strong>이때 주의할 점은 <code>Connection</code>을 현재 쓰레드에 바인딩하는 작업은 <code>startTransaction()</code>이 아니라 <code>2번째</code> <code>doBegin()</code> 내부에서 수행된다는 것입니다.</strong></p>
</li>
</ul>
<p><code>startTransaction()</code> 은 구현체의 <code>doBegin()</code>을 호출합니다.</p>
<ol start="4">
<li><p><code>JpaTransactionManager.doBegin()</code></p>
<pre><code class="language-java">
 protected void doBegin(Object transaction, TransactionDefinition definition) {
     JpaTransactionObject txObject = (JpaTransactionObject)transaction;
     if (txObject.hasConnectionHolder() &amp;&amp; !txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
         throw new IllegalTransactionStateException(&quot;Pre-bound JDBC Connection found! JpaTransactionManager does not support running within DataSourceTransactionManager if told to manage the DataSource itself. It is recommended to use a single JpaTransactionManager for all transactions on a single DataSource, no matter whether JPA or JDBC access.&quot;);
     } else {
         try {
             EntityManager em;
             if (!txObject.hasEntityManagerHolder() || txObject.getEntityManagerHolder().isSynchronizedWithTransaction()) {
                 em = this.createEntityManagerForTransaction();
                 if (this.logger.isDebugEnabled()) {
                     this.logger.debug(&quot;Opened new EntityManager [&quot; + em + &quot;] for JPA transaction&quot;);
                 }

                 txObject.setEntityManagerHolder(new EntityManagerHolder(em), true);
             }

             em = txObject.getEntityManagerHolder().getEntityManager();
             int timeoutToUse = this.determineTimeout(definition);
             Object transactionData = this.getJpaDialect().beginTransaction(em, new JpaTransactionDefinition(definition, timeoutToUse, txObject.isNewEntityManagerHolder()));
             txObject.setTransactionData(transactionData);
             txObject.setReadOnly(definition.isReadOnly());
             if (timeoutToUse != -1) {
                 txObject.getEntityManagerHolder().setTimeoutInSeconds(timeoutToUse);
             }

             if (this.getDataSource() != null) {
                 ConnectionHandle conHandle = this.getJpaDialect().getJdbcConnection(em, definition.isReadOnly());
                 if (conHandle != null) {
                     ConnectionHolder conHolder = new ConnectionHolder(conHandle);
                     if (timeoutToUse != -1) {
                         conHolder.setTimeoutInSeconds(timeoutToUse);
                     }

                     if (this.logger.isDebugEnabled()) {
                         this.logger.debug(&quot;Exposing JPA transaction as JDBC [&quot; + conHandle + &quot;]&quot;);
                     }

                     TransactionSynchronizationManager.bindResource(this.getDataSource(), conHolder);
                     txObject.setConnectionHolder(conHolder);
                 } else if (this.logger.isDebugEnabled()) {
                     this.logger.debug(&quot;Not exposing JPA transaction [&quot; + em + &quot;] as JDBC transaction because JpaDialect [&quot; + this.getJpaDialect() + &quot;] does not support JDBC Connection retrieval&quot;);
                 }
             }

             if (txObject.isNewEntityManagerHolder()) {
                 TransactionSynchronizationManager.bindResource(this.obtainEntityManagerFactory(), txObject.getEntityManagerHolder());
             }

             txObject.getEntityManagerHolder().setSynchronizedWithTransaction(true);
         } catch (TransactionException var9) {
             TransactionException ex = var9;
             this.closeEntityManagerAfterFailedBegin(txObject);
             throw ex;
         } catch (Throwable var10) {
             Throwable ex = var10;
             this.closeEntityManagerAfterFailedBegin(txObject);
             throw new CannotCreateTransactionException(&quot;Could not open JPA EntityManager for transaction&quot;, ex);
         }
     }
 }</code></pre>
<p>해당 <code>doBegin()</code> 메서드는 <code>JpaTransactionManager</code>에서 <code>JPA</code> 트랜잭션을 실제로 시작하는 로직의 핵심입니다.
이 코드 하나로 트랜잭션의 연결, 동기화, 커넥션 노출까지 모두 처리되기 때문에, 아주 중요한 코드입니다.
하나씩 살펴보겠습니다.</p>
</li>
</ol>
<blockquote>
<ol>
<li><code>JDBC 커넥션 충돌 검사</code></li>
</ol>
</blockquote>
<pre><code class="language-java">    if (txObject.hasConnectionHolder() &amp;&amp; !txObject.getConnectionHolder().isSynchronizedWithTransaction()) {
        throw new IllegalTransactionStateException(...);
    }</code></pre>
<p><code>Spring</code>에서 <code>JPA</code>와 <code>JDBC</code>를 동시에 같은 <code>DataSource</code>에서 사용하려 할 때 충돌이 발생할 수 있습니다
→ <code>JpaTransactionManager</code>를 쓰는 경우엔 <code>DataSourceTransactionManager</code>를 병행하면 안 됨</p>
<blockquote>
<ol start="2">
<li><code>EntityManager</code> 생성 또는 재사용</li>
</ol>
</blockquote>
<pre><code class="language-java">if (!txObject.hasEntityManagerHolder() || ...) {
    em = this.createEntityManagerForTransaction();
    txObject.setEntityManagerHolder(new EntityManagerHolder(em), true);
}</code></pre>
<p>없거나 재사용하면 안 되는 상황이면 새로 만들게 됩니다. 
<code>EntityManager</code>는 실제로 <code>JPA</code> 트랜잭션을 제어할 핵심 객체로, <code>true</code>는 이 <code>EntityManager</code>가 새로 만들어졌다는 표시입니다.</p>
<blockquote>
<ol start="3">
<li><code>리소스 바인딩</code></li>
</ol>
</blockquote>
<pre><code class="language-java">TransactionSynchronizationManager.bindResource(this.obtainEntityManagerFactory(), txObject.getEntityManagerHolder());</code></pre>
<p>트랜잭션 범위 안에서 동작하는 <code>DAO</code>나 <code>Repository</code>들이 <code>EntityManager</code>나 <code>Connection</code>을 자동으로 참조할 수 있도록 현재 쓰레드에 리소스를 등록합니다
<strong>주의 포인트: TransactionSynchronizationManager는 쓰레드 로컬 기반 → 반드시 커밋/롤백 이후 unbind 필요 (Spring이 자동 정리)</strong></p>
<blockquote>
<ol start="4">
<li><code>트랜잭션 동기화 완료 표시</code></li>
</ol>
</blockquote>
<pre><code class="language-java">txObject.getEntityManagerHolder().setSynchronizedWithTransaction(true);</code></pre>
<p>이 <code>EntityManager</code>는 현재 트랜잭션과 연결돼 있음을 명시합니다.</p>
<p>마지막 <code>doBegin()</code>을 호출하는 것으로 실제 트랜잭션이 시작되며 전체 흐름을 요약하면 아래와 같습니다.</p>
<h3 id="🔁-전체-흐름-요약-transactional-적용-후의-트랜잭션-처리-사이클">🔁 전체 흐름 요약: <code>@Transactional</code> 적용 후의 트랜잭션 처리 사이클</h3>
<pre><code class="language-text">@Transactional 메서드 호출
   ↓
AOP 프록시: TransactionInterceptor.invoke()
   ↓
invokeWithinTransaction() → 트랜잭션 전처리
   ↓
createTransactionIfNecessary()
   ↓
→ PlatformTransactionManager.getTransaction()
   ↓
→ AbstractPlatformTransactionManager.getTransaction()
   ↓
→ doGetTransaction() + handleExistingTransaction()
   ↓
→ startTransaction()
   ↓
→ ✅ doBegin() ← 실제 트랜잭션 시작
   ↓
비즈니스 로직 실행 (invocation.proceedWithInvocation())
   ↓
정상 종료 → commitTransactionAfterReturning()
         → PlatformTransactionManager.commit()
         → AbstractPlatformTransactionManager.commit()
         → doCommit() ← 진짜 커밋
OR
예외 발생 → completeTransactionAfterThrowing()
         → PlatformTransactionManager.rollback()
         → AbstractPlatformTransactionManager.rollback()
         → doRollback() ← 진짜 롤백
   ↓
TransactionInfo cleanup → ThreadLocal 해제</code></pre>
<h3 id="마무리하며">마무리하며</h3>
<p>지금까지 <code>@Transactional</code>이 동작하는 내부 구조를 따라가며, 트랜잭션이 어떻게 시작되고, 어떤 과정을 거쳐 커밋 또는 롤백되며, 마지막에 어떻게 정리되는지까지의 전체 흐름을 살펴보았습니다.</p>
<p>정리하자면, <code>PlatformTransactionManager</code>라는 추상화 덕분에 데이터 접근 기술이 변경되더라도 애플리케이션 코드를 수정하지 않고도 동일한 방식으로 트랜잭션을 관리할 수 있다는 점을 확인할 수 있었습니다.
또한 실제 코드를 따라가면서 트랜잭션 시작부터 종료까지 어떤 메서드가 어떤 순서로 호출되는지도 명확히 이해할 수 있었습니다.</p>
<p>다음 글에서는 이번 글에서 다루지 못한 리소스 동기화를 담당하는 <code>TransactionSynchronizationManager</code>의 역할과
트랜잭션 종료 시 실행되는 후처리 작업들에 대해 이어서 정리해보겠습니다.</p>