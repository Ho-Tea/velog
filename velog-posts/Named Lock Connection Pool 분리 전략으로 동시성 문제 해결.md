<blockquote>
<p><strong>본 프로젝트는 토스 페이먼츠 결제 API를 활용한 방탈출 예약 서비스입니다.</strong></p>
</blockquote>
<p>해당 글은 외부 결제 API를 통한 데이터 일관성 보장을 위해 도입한 트랜잭션 분리 전략 과정에서 예약 관련 사전 정보 저장 시 발생한 동시성 문제를 다룹니다. 이를 해결하기 위해 애플리케이션, 데이터베이스, 인프라 각 수준에서 적용할 동시성 제어 기법을 검토하고 선택한 기술적 접근 방식에 대해 설명합니다.</p>
<hr />
<h1 id="요약">[요약]</h1>
<p>동시성 문제 해결을 위해 <strong>애플리케이션, 데이터베이스, 인프라 각 수준에서 동시성 제어 기법</strong>을 직접 경험하고 검토한 결과, <code>MySQL Named Lock</code>을 활용한 분산 락 구현과 별도의 <code>Connection Pool</code> 분리를 도입하여 DB 부하를 최소화하는 전략을 선택하였습니다.</p>
<hr />
<h1 id="문제-상황">[문제 상황]</h1>
<blockquote>
<p>외부 결제 API를 사용할 때의 데이터 일관성을 보장하기 위해 <strong>파사드 패턴을 활용한 트랜잭션 분리 전략</strong>을 도입해 구현했었습니다. <strong>→ <a href="https://velog.io/@ho-tea/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EB%B6%84%EB%A6%AC-%EC%A0%84%EB%9E%B5%EC%9C%BC%EB%A1%9C-%EA%B0%95%ED%95%9C%EC%B5%9C%EC%A2%85-%EC%9D%BC%EA%B4%80%EC%84%B1-%EB%B3%B4%EC%9E%A5">🔗 결제 API - 트랜잭션 분리 전략으로 강한/최종 일관성 보장</a></strong></p>
</blockquote>
<p>그 과정에서, 예약 관련 사전 정보(<code>AdvanceReservation</code>)를 저장하는 과정에서 동시성 문제가 발생하였습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/023fdb68-9f34-4b95-b784-9ec6e6ea1ee5/image.png" /></p>
<hr />
<h2 id="문제-상황-시뮬레이션">[문제 상황 시뮬레이션]</h2>
<p>직접 작성한 테스트 코드로 동시에 100개의 요청이 들어오는 상황을 시뮬레이션한 결과, 예약은 최종적으로 <code>한 개만</code> 생성되어야 함에도 불구하고 <strong>10개의 예약이 생성되었습니다.</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/992a0e49-f711-4372-8502-b88ff3dceccb/image.png" /></p>
<h2 id="문제-상황-분석">[문제 상황 분석]</h2>
<p>상황은 다음과 같았습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/f0c8ee1e-9e4b-43ca-a4dc-e49c56f1a2f6/image.png" /></p>
<p><strong>하나의 <code>Reservation</code>에 대해 한 트랜잭션이 커밋되기 전에, 다른 트랜잭션이 동시에 <code>insert</code>를 수행하여 예약이 중복 생성되는 문제가 발생하였습니다.</strong></p>
<p>물론 <code>외부 결제 API</code> 측에서 별도의 처리를 해놓는다면, 뒤늦게 따라온 트랜잭션은 실패 처리될 수 있습니다. 하지만 <strong>외부 API의 응답 결과에만 의존하여 코드를 작성하는 것</strong>보다, <code>애플리케이션 레벨</code>, <code>데이터베이스 레벨</code>, <code>인프라 레벨</code>에서의 락이나 고유 제약 조건 등을 활용해 중복 예약이 발생하지 않도록 하는 것이 보다 안전한 설계라고 생각되었습니다.</p>
<hr />
<h1 id="해결-방안">[해결 방안]</h1>
<h2 id="❌-해결-방안-1--애플리케이션-수준에서의-동시성-제어">❌ [해결 방안 1 : 애플리케이션 수준에서의 동시성 제어]</h2>
<h3 id="synchronized-사용">[synchronized 사용]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/1a6e61fd-84ff-4e51-bff1-5c4d626792fe/image.png" /></p>
<h3 id="reentrantlock-사용">[ReentrantLock 사용]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/cdd67678-a546-4261-9033-853d04b6114e/image.png" /></p>
<p>위의 그림에서 보듯이, <code>synchronized</code>와 <code>ReentrantLock</code>을 사용한 경우 테스트가 성공적으로 통과하는 것을 확인할 수 있습니다.</p>
<p>이를 통해 애플리케이션 수준에서 동시성 제어가 수행되었음을 입증할 수 있었습니다.</p>
<hr />
<h3 id="애플리케이션-수준에서의-동시성-제어-한계점">[애플리케이션 수준에서의 동시성 제어 한계점]</h3>
<p><code>synchronized</code>에는 락 획득 대기 시 타임아웃 설정이나 공정성 정책을 지정할 수 없으며, 스레드가 락을 획득하지 못할 경우 무한정 대기하여 중단이나 타임아웃 제어가 어렵다는 단점이 있습니다.</p>
<p>이러한 문제점을 보완하기 위해 <code>ReentrantLock</code>을 사용하였습니다. <code>ReentrantLock</code>은 생성자에서 공정성 옵션을 지정할 수 있어 락 획득 대기 순서를 제어할 수 있으며, <code>tryLock()</code> 메서드를 사용해 지정된 시간 동안만 락 획득을 시도할 수 있어 락 획득 실패 시 대체 작업을 수행할 수 있는 장점을 제공합니다.</p>
<p><strong>하지만, 애플리케이션 레벨에서의 락은 다중 서버 환경에서는 적용하기 어려운 <code>치명적인 한계</code>가 있으므로, 데이터베이스에 락을 걸어 동시성 제어를 시도하였습니다.</strong></p>
<hr />
<h2 id="해결-방안-2--데이터베이스-수준에서의-동시성-제어">[해결 방안 2 : 데이터베이스 수준에서의 동시성 제어]</h2>
<h3 id="낙관적-락-vs-비관적-락">[낙관적 락 vs 비관적 락]</h3>
<p>낙관적 락은 데이터베이스에 이미 영속화되어 버전 필드(<code>@Version</code>)가 존재하는 엔티티에서 동시성 충돌을 감지하는 데 사용되므로, 새로운 엔티티 생성 시점에서는 적용할 수 없었습니다. 이러한 이유로, 낙관적 락 대신 비관적 락을 선택하여 구현하였습니다.</p>
<hr />
<h3 id="❌-비관적-락-사용">❌ [비관적 락 사용]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/314c9146-ede3-42ad-a566-b9e1f7f2716a/image.png" /></p>
<p><strong>위 그림에서 보듯이, 비관적 락을 사용한 경우 또한 테스트는 정상적으로 통과했습니다.</strong></p>
<p>그러나 특정 레코드의 존재 여부를 확인하기 위해 전체 테이블을 조회하는 쿼리에 비관적 락을 적용하게되면, 해당 테이블의 <strong>모든 레코드에 락이 걸려 다른 트랜잭션들이 읽기나 쓰기 작업을 수행할 수 없게 됩니다.</strong></p>
<p><strong>이는 동시성과 성능에 심각한 영향을 미칠 수 있다고 판단하였습니다.</strong></p>
<p>(많은 사용자가 동시에 접근하는 환경에서 한 트랜잭션이 전체 테이블에 락을 걸고 있는 동안, 다른 트랜잭션들은 락 해제가 될 때까지 대기 상태에 빠져 응답 지연이나 병목 현상이 발생합니다.)</p>
<hr />
<h3 id="❌-유니크-제약조건으로-동시성-제어">❌ [유니크 제약조건으로 동시성 제어]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/d28e2c71-4e63-40d9-b900-915aa9468e5e/image.png" /></p>
<p>락은 어떻게 됐든 병목지점을 만드는 것이기에 트레이드 오프라고 생각되었습니다. 따라서 DB 부하를 최소화하고 다른 트랜잭션에 미치는 영향을 줄일 수 있는 방법으로 <strong>유니크 제약 조건</strong>을 통해 동시성 제어를 시도하였습니다.</p>
<p>데이터베이스 엔진은 유니크 인덱스를 통해 중복 검사를 효율적으로 처리하므로, 애플리케이션에서 비관적 락이나 분산 락을 직접 구현하는 것보다 오버헤드가 적고 성능에 미치는 영향이 작다고 판단했습니다.</p>
<p>유니크 제약 조건은 데이터베이스 자체에서 처리되므로 여러 애플리케이션 인스턴스가 동일한 데이터베이스를 사용할 때도 일관된 데이터 무결성을 유지할 수 있었습니다. </p>
<h3 id="유니크-인덱스-데드락-발생-가능성">[유니크 인덱스 데드락 발생 가능성]</h3>
<blockquote>
<p><strong>ReservationService.java</strong></p>
</blockquote>
<pre><code class="language-java">@Transactional
public Reservation saveAdvanceReservationPayment(
        LoginMember loginMember,
        ReservationRequest reservationRequest,
        PaymentRequest paymentRequest
) {
    Reservation reservation = getReservation(loginMember.getId(), reservationRequest, ReservationStatus.ADVANCE_BOOKED);

    Reservations reservations = new Reservations(reservationRepository.findAll());
    if (reservations.hasSameReservation(reservation)) {
        throw new RoomescapeException(
                DUPLICATE_RESERVATION,
                reservationRequest.date(),
                reservationRequest.themeId(),
                reservationRequest.timeId());
    }
    reservationRepository.save(reservation);
    paymentRepository.save(new Payment(reservation, paymentRequest.paymentKey(), paymentRequest.amount()));
    return reservation;
}</code></pre>
<p>하지만 <code>saveAdvanceReservationPayment()</code>가 호출된 이후 아래와 같이 동일한 트랜잭션 내에서 <code>외부 결제 API</code>를 호출하는 형식으로 구성되어 있기에, </p>
<p><strong>외부 결제 API 호출이 실패하게 된다면 데드락의 발생 가능성이 높아집니다.</strong></p>
<blockquote>
<p><strong>ReservationApplicationService.java</strong></p>
</blockquote>
<pre><code class="language-java">@Transactional(propagation = Propagation.REQUIRES_NEW, timeout = 12)
public ReservationPaymentResult saveAdvanceReservationPayment(LoginMember loginMember, ReservationPaymentRequest reservationPaymentRequest) {
    Reservation reservation = reservationService.saveAdvanceReservationPayment(loginMember, reservationPaymentRequest.toReservationRequest(), reservationPaymentRequest.toPaymentRequest());
    **PaymentResult paymentResult = paymentClient.purchase(reservationPaymentRequest.toPaymentRequest()); // 외부 결제 API 호출**
    return new ReservationPaymentResult(reservation, paymentResult);
}</code></pre>
<h3 id="유니크-인덱스-데드락-발생-상황">[유니크 인덱스 데드락 발생 상황]</h3>
<blockquote>
<p><strong>Transaction1이 insert를 넣고 Transaction2가 insert를 넣고 Transaction3이 insert를 넣은 상황</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/89dbb94d-bfe7-421e-a717-39d18b4872c1/image.png" /></p>
<blockquote>
<p><strong>해당 상황에서 트랜잭션 1번을 ROLLBACK 할 경우 데드락이 발생</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/b6641b24-d524-49d0-9ef4-ec9cb7fa2a43/image.png" /></p>
<p><strong>외부 결제 API의 상태는 저희가 제어할 수 없는 영역이며, 그로 인해 응답의 성공과 실패를 항상 보장할 수 없기에 위와 같은 데드락의 발생 가능성이 충분히 존재한다고 생각되었습니다.</strong></p>
<p><strong>따라서, <code>레코드에 락을 것</code>과 <code>고유 제약 조건을 거는 것</code> 대신 MySQL에서 지원하는 메타데이터 기반의 네임드 락(named lock)을 사용하여 동시성 제어를 시도하였습니다.</strong></p>
<hr />
<h3 id="✅-mysql-분산락---named-lock-사용">✅ [MySQL 분산락 - Named Lock 사용]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/04545c07-494b-4af4-8b5c-761e8d4343da/image.png" /></p>
<p><strong>네임드 락을 사용한 경우에도 테스트가 정상적으로 통과함을 확인했습니다.</strong></p>
<blockquote>
<p>아직 데이터베이스에 저장되지 않은 <code>Reservation</code> 엔티티는 식별할 고유 키가 없으므로, 예약 날짜, 테마 ID, 그리고 타임 ID를 조합하여 고유한 키를 생성하였습니다.</p>
</blockquote>
<p>서비스 계층의 비즈니스 로직에서는 새로운 트랜잭션을 위해 <code>REQUIRES_NEW</code>옵션을 적용하고 있는 상태에서 비즈니스 로직에서 사용하는 <code>Connection Pool</code>과 동일한 <code>Connection Pool</code>을 <code>Lock</code> 점유를 위해 사용하게 된다면 <code>HikariCP Maximum Pool Size</code> 로 인해 <strong>커넥션 고갈 문제</strong>가 발생하게 됩니다.</p>
<hr />
<h3 id="✅-mysql-분산락---named-lock-connection-pool-분리">✅ [MySQL 분산락 - Named Lock Connection Pool 분리]</h3>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/a7219a7b-c6dc-44d5-8c04-e7901e90c9af/image.png" /></p>
<p><strong>따라서 Lock 로직이 사용하는 Connection Pool 과 비즈니스 로직이 사용하는 Connection Pool 을 분리하여 구성하였습니다.</strong> </p>
<p><strong>락과 관련된 작업이 별도의 Connection을 통해 독립적으로 처리되어, 락 획득이나 해제와 같은 작업으로 인해 비즈니스 로직에서 사용하는 Connection Pool이 소진되거나 지연되는 상황을 방지할 수 있었습니다.</strong></p>
<hr />
<h2 id="❌-해결-방안-3--인프라-수준에서의-동시성-제어">❌ [해결 방안 3 : 인프라 수준에서의 동시성 제어]</h2>
<h3 id="redis-분산락">[Redis 분산락]</h3>
<p><strong>네임드 락을 사용하면, 데이터베이스에 일시적인 락 정보가 저장되고, 락을 획득하고 해제하는 쿼리가 매번 발생하여 불필요한 DB 부하가 초래될 수 있습니다.</strong> 이에 <code>Redis</code>를 활용한 분산 락 방식도 고려해 보았습니다. </p>
<p><code>Redis</code>는 메모리를 사용하기 때문에 락을 빠르게 획득하고 해제할 수 있으며, 휘발성 특성 덕분에 네임드 락이 가지는 문제를 일부 해결할 수 있습니다. 하지만, <strong><code>Redis</code>를 이용하여 분산락을 구현하게 되면 인프라 구축에 대한 비용이 발생할 뿐만 아니라 인프라 구축 후 유지보수에 대한 비용 또한 발생하게 된다고 생각합니다.</strong> </p>
<p><code>MySQL</code>은 프로젝트 초기부터 <code>RDBMS</code>로 사용해오고 있었기 때문에 인프라 구축 및 유지보수에 대한 추가 비용이 들지 않았고, 분산락의 사용량이 추가적인 비용을 들일 만큼 많지 않았기 때문에 <code>MySQL</code> 을 이용하게 되었습니다. </p>
<p><strong>동시성 문제가 정말 빈번하게 발생하는지, 그리고 <code>Redis</code>를 도입할 충분한 이유가 있는지 고민한 결과, 그 필요성이 크지 않다고 판단했습니다.</strong></p>
<h1 id="결론">[결론]</h1>
<p>본 프로젝트에서는 토스 페이먼츠 결제 API를 활용한 방탈출 예약 서비스 구현 과정에서, 예약 관련 사전 정보 저장 시 발생한 동시성 문제를 해결하기 위해 여러 수준(애플리케이션, 데이터베이스, 인프라)의 동시성 제어 기법을 검토하였습니다. 초기에는 애플리케이션 수준에서의 synchronized 및 ReentrantLock을 통한 동시성 제어를 시도했으나, <strong>다중 서버 환경에서의 한계로 인해 데이터베이스 수준에서의 접근 방식으로 전환하였습니다.</strong></p>
<p>데이터베이스에서는 비관적 락, 유니크 제약 조건 등 여러 기법을 비교했으며, 특히 유니크 제약 조건은 성능에 미치는 부하를 최소화할 수 있는 장점이 있으나, 외부 결제 API 호출과 결합된 복합 로직에서 <strong>데드락 발생 가능성이 존재함을 확인했습니다.</strong> 이러한 문제를 해결하기 위해 MySQL의 <code>Named Lock</code> 기능을 활용하여 분산 락을 구현하고, 이와 동시에 비즈니스 로직과 락 로직이 사용하는 <code>Connection Pool</code>을 분리하여 <strong>커넥션 고갈 문제를 방지하였습니다.</strong></p>
<p><strong>최종적으로, 여러 동시성 제어 방안 중 인프라에 추가 비용을 들이지 않고 기존 RDBMS 환경 내에서 효율적으로 문제를 해결할 수 있는 Named Lock 기반의 분산 락을 선택하였습니다.</strong></p>