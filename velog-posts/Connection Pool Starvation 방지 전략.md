<blockquote>
<p><strong>본 프로젝트는 토스 페이먼츠 결제 API를 활용한 방탈출 예약 서비스입니다.</strong></p>
</blockquote>
<p>해당 글은 아래의 <code>3가지 조건</code>에 기반하여 결제 API 연동 과정에서 트랜잭션 관리, 원자성 보장, 데이터 무결성 확보 등을 위해 선택한 기술적 접근 방식에 대해 설명합니다. </p>
<ul>
<li><input checked="" disabled="" type="checkbox" /> <strong>조건 1 : 결제 도메인의 경우, 돈이 오가는 민감한 트랜잭션을 처리하므로 데이터의 정확성과 일관성이 가장 중요하다.</strong></li>
</ul>
<p><strong>→ <code>트랜잭션 분리 전략</code> = <a href="https://velog.io/@ho-tea/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EB%B6%84%EB%A6%AC-%EC%A0%84%EB%9E%B5%EC%9C%BC%EB%A1%9C-%EA%B0%95%ED%95%9C%EC%B5%9C%EC%A2%85-%EC%9D%BC%EA%B4%80%EC%84%B1-%EB%B3%B4%EC%9E%A5">🔗 결제 API - 트랜잭션 분리 전략으로 강한/최종 일관성 보장</a></strong></p>
<ul>
<li><input checked="" disabled="" type="checkbox" /> <strong>조건 2 : 방탈출을 예약하는 과정에서 사용자는 출금 결과와 예약 결과를 알고 넘어가야하기 때문에 동기로 구성해야 한다.</strong></li>
</ul>
<p><strong>→ <code>모든 작업을 동기 방식으로 구성</code>  = <a href="https://velog.io/@ho-tea/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EB%B6%84%EB%A6%AC-%EC%A0%84%EB%9E%B5%EC%9C%BC%EB%A1%9C-%EA%B0%95%ED%95%9C%EC%B5%9C%EC%A2%85-%EC%9D%BC%EA%B4%80%EC%84%B1-%EB%B3%B4%EC%9E%A5">🔗 결제 API - 트랜잭션 분리 전략으로 강한/최종 일관성 보장</a></strong></p>
<ul>
<li><strong>조건 3 : 사용자는 예약이 즉시 완료되기를 기대하므로 Time Out을 설정해야 한다.</strong></li>
</ul>
<hr />
<h1 id="요약">[요약]</h1>
<blockquote>
<p>해당 글에서는 <code>조건 3</code>을 충족하는 과정을 담고 있으며, <code>조건 1</code>과 <code>조건 2</code>를 충족하는 과정은 
<a href="https://velog.io/@ho-tea/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EB%B6%84%EB%A6%AC-%EC%A0%84%EB%9E%B5%EC%9C%BC%EB%A1%9C-%EA%B0%95%ED%95%9C%EC%B5%9C%EC%A2%85-%EC%9D%BC%EA%B4%80%EC%84%B1-%EB%B3%B4%EC%9E%A5">🔗 결제 API - 트랜잭션 분리 전략으로 강한/최종 일관성 보장</a>에서 확인 가능합니다.</p>
</blockquote>
<p><strong>조건 3 : 사용자는 예약이 즉시 완료되기를 기대하므로 Time Out을 설정해야 한다.</strong></p>
<p><strong>결론적으로 <code>Transaction timeout</code>과 <code>Http timeout</code>을 설정하는것으로 해결하였습니다.</strong></p>
<hr />
<h1 id="문제-상황">[문제 상황]</h1>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/d3ea0e28-0959-4fbc-a411-cb7a8c9a302b/image.png" /></p>
<p><strong>외부 결제 API와 예약/결제 사전 정보를 DB에 저장하는 작업</strong>을 하나의 트랜잭션으로 묶게 되면, 외부 API 호출에 소요되는 지연 시간 동안 내부 DB 커넥션이 계속해서 점유되므로 커넥션 고갈 문제가 발생할 수 있습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/3fedc6bc-798d-474f-9b0a-2f4c578306e5/image.png" /></p>
<p>이러한 상황은 내부 DB 접근에 제약을 주어 <strong>전체 시스템의 성능 저하 및 장애로 이어질 위험</strong>이 있다고 판단하였습니다.</p>
<p>또한 <strong>사용자는 예약이 즉시 완료되기를 기대하지만,</strong> 지연 시간 동안 사용자 경험이 저하되어 불만이 쌓이고, 이는 궁극적으로 고객 이탈로 이어질 수 있다고 판단하였습니다.</p>
<hr />
<h1 id="해결-방안">[해결 방안]</h1>
<ul>
<li><strong>Transaction timeout</strong> : 내부 DB 작업의 시간 제한을 통해 데이터 무결성을 보장</li>
<li><strong>Http timeout :</strong> 외부 시스템과의 통신에서 응답 지연을 방지</li>
</ul>
<h2 id="✅-transaction-timeout">✅ [Transaction Timeout]</h2>
<blockquote>
<p>외부 결제 API 호출 전에 <strong>DB에 예약/결제 사전 정보를 저장하는 작업과 외부 결제 API 호출은 하나의 트랜잭션</strong>으로 처리하고, 그 후 <strong>예약/결제 상세 정보를 저장하는 작업</strong>은 <code>별도의 트랜잭션으로 분리</code></p>
</blockquote>
<pre><code class="language-java">// ReservationFacade.java

public ReservationPaymentResponse saveReservationPayment(
        LoginMember loginMember,
        ReservationPaymentRequest reservationPaymentRequest
) {
    ReservationPaymentResult reservationPaymentResult = reservationApplicationService.saveAdvanceReservationPayment(loginMember, reservationPaymentRequest);
    try {
        return reservationApplicationService.saveDetailedReservationPayment(reservationPaymentResult.reservation(), reservationPaymentResult.paymentResult());
    } catch (Exception e) {
        return new ReservationPaymentResponse(
                ReservationResponse.from(reservationPaymentResult.reservation()),
                PaymentResponse.from(reservationPaymentResult.paymentResult()));
    }
}

// ReservationApplicationService.java

**// [사전 예약 정보 &amp; 결제 정보 DB 저장]**
**@Transactional(propagation = Propagation.REQUIRES_NEW, timeout = 6)**
public ReservationPaymentResult saveAdvanceReservationPayment(LoginMember loginMember, ReservationPaymentRequest reservationPaymentRequest) {
    Reservation reservation = reservationService.saveAdvanceReservationPayment(loginMember, reservationPaymentRequest.toReservationRequest(), reservationPaymentRequest.toPaymentRequest());
    PaymentResult paymentResult = paymentClient.purchase(reservationPaymentRequest.toPaymentRequest());
    return new ReservationPaymentResult(reservation, paymentResult);
}

**// [상세 예약 정보 &amp; 결제 정보 DB 저장]**
**@Transactional(propagation = Propagation.REQUIRES_NEW, timeout = 1)**
public ReservationPaymentResponse saveDetailedReservationPayment(
        Reservation reservation,
        PaymentResult paymentResult
) {
    return reservationService.confirmReservationPayment(reservation, paymentResult);
}</code></pre>
<p>각 트랜잭션에 <code>timeout</code>을 설정함으로써 비정상적인 지연 상황에서 자원 점유를 제한하고, 시스템 전체의 <strong>응답성</strong>을 유지할 수 있습니다.</p>
<blockquote>
<p>데이터베이스에 여러 레코드를 삽입하고 외부 결제 시스템과 통신하는 등 상대적으로 시간이 소요될 수 있는 작업 → <code>timeout 6초</code>
응답 지연 없이 빠르게 작업이 완료되어야 하는 작업 → <code>timeout 1초</code></p>
</blockquote>
<h2 id="✅-http-timeout">✅ [Http Timeout]</h2>
<pre><code class="language-java">// RestClientConfig.java

// 커넥션 풀 매니저 생성 메서드
private PoolingHttpClientConnectionManager getPoolingHttpClientConnectionManager() {
    // 커넥션 구성: **커넥션 생성 및 소켓 타임아웃을 설정**
    ConnectionConfig connectionConfig = ConnectionConfig.custom()
            .setConnectTimeout(connectionTimeOut)
            .setSocketTimeout(socketTimeOut)
            .build();

    return PoolingHttpClientConnectionManagerBuilder.create()
            .setDefaultConnectionConfig(connectionConfig)
            .build();
}

// 요청 타임아웃 설정 메서드
private RequestConfig getRequestConfig() {
    **// 응답 수신 시간 제한(read timeout)을 설정**
    return RequestConfig.custom()
            .setResponseTimeout(readTimeOut)
            .build();
}

// HttpClient 생성 메서드
private CloseableHttpClient getHttpClient(PoolingHttpClientConnectionManager connManager, RequestConfig requestConfig) {
    return HttpClients.custom()
            .setConnectionManager(connManager) // 커넥션 풀 매니저를 사용하여 커넥션 재사용
            .setDefaultRequestConfig(requestConfig) // 요청 관련 타임아웃 설정 적용
            **// 재시도 전략 설정: 최대 3회 재시도, 1초 간격으로 재시도**
            .setRetryStrategy(new DefaultHttpRequestRetryStrategy(3, Timeout.ofSeconds(1)))
            .build();
}</code></pre>
<p>지연시간을 제한하기 위해 <code>Connection Timeout</code>, <code>Socket Timeout</code>, <code>Read Timeout</code>을 설정하여 정해진 응답 시간 내에 응답이 오지 않으면 호출이 실패처리하는 것으로 구성하였습니다. 또한, 일시적인 네트워크 오류나 서버 과부하로 인해 외부 API 호출이 실패할 경우를 대비하여, 지정된 횟수만큼 재시도하는 <code>Retry</code> 로직을 추가로 구성하였습니다.</p>
<blockquote>
<p><code>ConnectionTimeOut</code> : 3초</p>
<ul>
<li>대부분의 시스템에서 <code>InitRTO</code>의 값이 1초로 설정되어 있다고 판단 (<a href="https://datatracker.ietf.org/doc/html/rfc6298">RFC 6298</a>)</li>
<li><code>Syn</code> 패킷 유실, <code>Syn + Ack</code> 패킷 유실, <code>Ack</code> 패킷 유실 과 같이 연결이 지연되어 실패하는 경우를 3가지로 판단하여 3초로 설정</li>
</ul>
<p><code>ReadTimeOut</code> : 30초</p>
<ul>
<li>해당 부분은 <a href="https://docs.tosspayments.com/resources/glossary/timeout">토스 페이먼츠</a>에 기술되어있는 것을 확인하여 30초로 구성</li>
</ul>
<p><code>SocketTimeOut</code> : 2초</p>
<ul>
<li>패킷 하나가 유실되고 다시 재전송되기까지의 시간을 고려하여 구성</li>
</ul>
</blockquote>
<hr />
<h1 id="결론">[결론]</h1>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/5f0b41fa-0991-4be1-bd66-1cb57943cf0b/image.png" /></p>
<p>외부 API와 내부 DB 작업을 하나의 트랜잭션으로 묶게 되면, 외부 API 호출에 소요되는 지연 시간 동안 내부 DB 커넥션이 계속 점유되어 커넥션 고갈 문제가 발생할 위험이 있습니다. 또한, <strong>사용자는 예약이 즉시 완료되기를 기대하지만</strong>, 이러한 지연으로 인해 응답 시간이 길어지면 사용자 경험이 크게 저하될 수 있습니다.</p>
<p>이를 방지하기 위해, </p>
<ul>
<li>각 트랜잭션에 <code>timeout</code>을 설정함으로써 비정상적인 지연 상황에서 자원 점유를 제한하고, 시스템 전체의 <strong>응답성</strong>을 유지할 수 있습니다.</li>
</ul>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/fcb8c6b0-f923-4152-9daa-af3610674a51/image.png" /></p>
<ul>
<li><code>Connection Timeout</code>, <code>Socket Timeout</code>, <code>Read Timeout</code>을 설정하여 각 단계의 지연 시간을 제한했습니다. 또한, 일시적인 네트워크 오류나 서버 과부하로 인해 외부 API 호출이 실패할 경우를 대비하여, 지정된 횟수만큼 재시도하는 <code>Retry</code> 로직을 추가로 구성하였습니다.</li>
</ul>