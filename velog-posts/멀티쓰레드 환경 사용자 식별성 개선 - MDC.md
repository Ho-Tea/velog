<blockquote>
<p><strong>본 프로젝트는 Spring Boot와 MySQL을 활용한 기록 프로젝트 입니다.</strong></p>
</blockquote>
<p>해당 글은 기록 프로젝트를 진행하면서 사용자 로그가 순차적으로 기록되지 않아 겪었던 사용자 식별에 대한 어려움을 해결하고자 선택한 기술적 접근 방식에 대해 설명합니다. </p>
<hr />
<h1 id="요약">[요약]</h1>
<p>여러 사용자가 동시에 애플리케이션에 접근하면 각기 다른 쓰레드를 사용하기 때문에 동일한 요청에 대한 로그가 순차적으로 쌓이는 것이 아닌 순서없이 쌓이게 되는 문제를 <code>MDC</code> 적용과 <code>ArgumentResolver</code> 사용자 식별 로깅을 통해 해결하였습니다.</p>
<hr />
<h1 id="문제-상황">[문제 상황]</h1>
<p>멀티 쓰레드 환경에서는 요청이 동시에 처리되는데, 여러 사용자가 동시에 애플리케이션에 접근하면 각기 다른 쓰레드를 사용하기 때문에 동일한 요청에 대한 로그가 순차적으로 쌓이는 것이 아닌 순서없이 쌓이게 됩니다.</p>
<p>따라서 사용자별 로그가 순차적으로 기록되지 않는 문제와 어떠한 사용자가 요청을 한것인지 식별하기 어려운 문제로 인해 요청별 로그 추적에 어려움을 겪었습니다.</p>
<blockquote>
<p>아래와 같이 테스트를 수행하였을 때에도 <code>문제 상황</code>을 명확히 인지할 수 있습니다.</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/a6bc5134-f470-4500-b55f-636dfb40e308/image.png" /></p>
<ul>
<li>사용자 별 로그가 순차적으로 기록되지 않아 요청 별 로그를 추적하기 쉽지 않습니다. </li>
<li><em>→ <code>MDC</code> 적용으로 해결 시도*</em></li>
<li>어떠한 사용자가 어느 시점에 로그인을 시도한지 식별할 수 없습니다. </li>
<li><em>→ <code>Argument Resolver</code>에서 사용자 식별 로깅 추가로 해결 시도*</em></li>
</ul>
<hr />
<h1 id="해결-방안">[해결 방안]</h1>
<h2 id="✅-mdc-적용">✅ [MDC 적용]</h2>
<blockquote>
<p><strong>LoggingFilter.java</strong></p>
</blockquote>
<pre><code class="language-java">@Slf4j
@Component
public class LoggingFilter extends OncePerRequestFilter {

    // 로그 메시지에 사용할 고정 키, MDC(Mapped Diagnostic Context)에 저장됩니다.
    private static final String IDENTIFIER = &quot;request_id&quot;;

    // 필터에서 제외할 URL 패턴들을 정의한 화이트리스트입니다.
    private static final List&lt;String&gt; WHITE_LIST = List.of(
            &quot;/h2-console/**&quot;,
            &quot;/favicon/**&quot;,
            &quot;/swagger-ui/**&quot;,
            &quot;/v3/api-docs/**&quot;,
            &quot;/metrics&quot;,
            &quot;/actuator/**&quot;);

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        // 요청 처리 시간 측정을 위해 StopWatch를 시작합니다.
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        // 각 요청에 대해 고유한 식별자(UUID)를 생성하여 MDC에 저장합니다.
        MDC.put(IDENTIFIER, UUID.randomUUID().toString());

        // 요청 헤더에서 Authorization 토큰을 가져옵니다.
        String token = request.getHeader(HttpHeaders.AUTHORIZATION);

        try {
            // 다음 필터 혹은 최종 리소스로 요청을 전달합니다.
            filterChain.doFilter(request, response);
        } finally {
            // 요청 처리가 끝난 후 StopWatch를 중지합니다.
            stopWatch.stop();

            // 요청 로그를 기록합니다.
            // 로그에 응답 상태 코드, HTTP 메서드, 요청 URI, 토큰 존재 여부, 처리 시간(ms)을 포함합니다.
            log.info(LogForm.REQUEST_LOGGING_FORM,
                    response.getStatus(),
                    request.getMethod(),
                    request.getRequestURI(),
                    tokenExists(token),
                    stopWatch.getTotalTimeMillis());

            // MDC의 내용을 초기화하여, 다른 요청에 영향을 주지 않도록 합니다.
            MDC.clear();
        }
    }
    ...
}</code></pre>
<blockquote>
<p><strong>xxx-appender.xml</strong></p>
</blockquote>
<pre><code class="language-java">&lt;encoder&gt;
    &lt;pattern&gt;[%d{yyyy-MM-dd HH:mm:ss}:%-3relative] [%thread] [request_id=%X{request_id:-startup}] %-5level - %msg%n&lt;/pattern&gt;
&lt;/encoder&gt;</code></pre>
<blockquote>
<p><strong>예시 로그</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/34cbc620-1877-4789-8c72-8813e67d946f/image.png" /></p>
<p>이와 같이 <strong><code>MDC</code></strong>를 적용하면서 사용자별 로그를 구분해 각 사용자의 행동을 추적할 수 있으며,</p>
<p>다중 사용자가 동시에 요청을 보낼 때 각 사용자의 컨텍스트가 분리되어 로그가 혼합되지 않는다는 장점을 경험할 수 있었습니다.</p>
<hr />
<h2 id="✅-argument-resolver-사용자-식별">✅ [Argument Resolver 사용자 식별]</h2>
<blockquote>
<p><strong>예시 로그</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/0e3c62a1-9ab7-4cf4-bf30-fc4062712ec0/image.png" /></p>
<p><strong><code>MDC</code></strong>와 더불어 <strong><code>Argument Resolver에서 사용자를 식별</code></strong>하고 로깅을 수행하게 되면,</p>
<p>사용자 요청이 들어올 때마다 Argument Resolver가 먼저 실행되어 각 요청이 어느 사용자인지 명확히 식별할 수 있습니다.</p>
<p>(이때 요청의 고유 ID가 MDC를 통해 저장되므로, 특정 사용자의 요청을 명확하게 추적할 수 있습니다.)</p>
<hr />
<h1 id="결론">[결론]</h1>
<p>실제 <code>Grafana</code>를 통해 로깅을 모니터링 했을 때 아래와 같이 <strong><code>MDC</code></strong>와 <strong><code>Argument Resolver</code></strong> 로깅을 통한 사용자 식별이 적용된 것을 확인해 볼 수 있습니다.</p>
<p>이러한 로그 데이터를 분석함으로써 특정 사용자가 자주 요청하는 API 엔드포인트, 로그인 시도 실패 기록과 패턴, 특정 사용자에게서 발생하는 예외 상황, 사용자별 응답 시간 및 성능 문제 등을 확인할 수 있습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/bb8bab3b-b25b-44d6-8c2f-886a6a39f531/image.png" /></p>
<blockquote>
<p><strong>동일한 쓰레드를 사용하더라도, request_id가 다르므로 서로 다른 요청</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/ee2a624f-3658-4667-8a8d-f441ff49037d/image.png" /></p>
<blockquote>
<p><strong>로그 상세 추적을 위한 Panel</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/736a515a-8f0f-4a05-b18c-da3c09582dee/image.png" /></p>
<p>이와 같이 동일한 <strong><code>request_id</code></strong>를 가진 요청들을 별도로 분리하여 로그를 상세하게 추적할 수 있는 대시보드를 구성하였습니다.</p>
<p>이를 통해 사용자의 행동을 분석하여 서비스 최적화 및 개인화된 서비스 제공에 필요한 데이터로 활용할 수도 있습니다. 또한, 보안 측면에서 의심스러운 요청을 특정 사용자와 연결하여 신속하게 탐지할 수 있습니다.</p>