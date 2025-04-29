<blockquote>
<p><strong>본 프로젝트는 Spring Boot와 MySQL을 활용한 기록 프로젝트 입니다.</strong></p>
</blockquote>
<p>해당 글은 현재 운영 중인 서버가 트래픽 급증 시에도 안전하게 서비스를 제공할 수 있도록 인스턴스 단위의 성능 튜닝 및 최적화 과정에서 선택한 기술적 접근 방식을 설명합니다.</p>
<hr />
<h1 id="요약">[요약]</h1>
<p>현재 운영 중인 서버는 <code>Scale-Out</code>과 <code>Scale-Up</code> 방식에 한계가 존재합니다. </p>
<p>트래픽이 급증하는 상황에 대비하여 애플리케이션의 성능을 튜닝하여 안전성있는 서비스를 제공하는 것이 필수적이라고 판단했습니다. 따라서, <strong>인스턴스 한 대 기준</strong>으로 핵심기능이 안정적으로 서비스 될 수 있게 설정하는 과정을 거치게 되었습니다.</p>
<hr />
<h1 id="tomcat과-hikaricp의-설정값만을-바꾸면서-테스트를-진행한-이유">[Tomcat과 HikariCP의 설정값만을 바꾸면서 테스트를 진행한 이유]</h1>
<p><strong>성능 개선에서 무분별한 변경은 원인 분석을 어렵게 만든다고 생각했습니다.</strong></p>
<p>저희는 우선, 애플리케이션의 응답 지연이 <code>DB 커넥션 풀</code>과 <code>서버 쓰레드 처리량</code>에 기인할 수 있다는 가설을 세웠고, 이에 따라 <strong>Tomcat</strong>의 최대 스레드 수, <strong>HikariCP</strong>의 최소/최대 커넥션 수와 같은 <strong>I/O 병목 지점에 직접적인 영향을 주는 설정값부터 조정</strong>해 테스트를 진행했습니다.</p>
<p>정해진 일정에 차질이 가지 않도록 <strong>하나의 요인을 기준으로 우선 테스트를 진행하였고</strong>, 추가적인 성능 테스트를 위해 APM, 캐시 도입 등 추가적인 영역으로 테스트 범위를 확장하려는 계획을 갖고 있습니다. </p>
<hr />
<h1 id="과정">[과정]</h1>
<h2 id="테스트-시나리오-선정-및-ngrinder-setting">[테스트 시나리오 선정 및 Ngrinder Setting]</h2>
<blockquote>
<p>Ngrinder Setting -&gt; Vuser 1000 (process 1, thread 1000)
test 기간: 3m</p>
</blockquote>
<p>Firebase Analytics 을 확인해 보았을 때 조회 API를 호출하는 화면의 비중이 높았습니다.</p>
<p>그로인해 로그인, 추억 조회, 기록 조회 API 를 Think Time이 적용된 하나의 테스트 시나리오로 묶어 스크립트를 작성, 각 테스트 데이터는 100만건을 삽입한 후 Ngrinder를 통해 부하 테스트를 진행하였습니다.</p>
<p>또한 활성 사용자수와 현재 서비스 크기를 고려했을때, 약 1000명의 사용자가 요청을 보내는 상황으로 가정하였습니다.</p>
<blockquote>
<p><strong>Firebase Analytics</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/05707fce-6ec1-451e-bd55-d6dd3975cb0e/image.png" /></p>
<hr />
<h2 id="초기-값default-value으로-성능-측정">[초기 값(Default Value)으로 성능 측정]</h2>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/1ec59dc0-a088-49e7-aad3-ba3215212394/image.png" /> <strong>Default Value</strong>| <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/43a51d92-d595-4203-9df9-f6d75f65ea70/image.png" /> <strong>Default Value 성능 측정 결과</strong>
|---|---</p>
<p>팀 프로젝트에서 Spring Boot(3.3.1)를 사용하였기에 위와 같은 Default 값들이 설정되어있었습니다.</p>
<p>각 설정들을 하나씩 <strong><code>독립 변인</code></strong>으로 설정한 후, 다른 설정들은 <strong><code>통제 변인</code></strong>으로 설정, 그로 인해 도출되는 결과 TPS와 성공 응답 비율을 <strong><code>종속 변인</code></strong>으로 설정하여 테스트를 진행하였습니다.</p>
<h2 id="tomcat-max-connections">[Tomcat-Max-Connections]</h2>
<p>1000명 기준 평균 TPS가 30인 것에 비해 <code>Max Connections</code>이 과도하게 크다고 생각되었습니다.</p>
<p>따라서 연결할 수 있는 개수를 줄여 리소스 낭비를 줄이는 방식을 선택하였습니다. 대신, <code>Connection</code>을 얻지 못해서 테스트 실패율이 증가하였습니다.</p>
<p>테스트 실패율과 TPS를 두고 저울질 한다면, 테스트 실패율이 더 낮은 것이 중요하다고 생각해서 <code>Connection</code>의 개수를 2048개로 선택하게 되었습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/fa08118b-139e-463e-92e1-71e2e1be4c86/image.png" /> <strong>[Max-Connection = 8192]</strong>| <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/f9890d86-6050-43da-a71b-850aa6ba91bd/image.png" /> <strong>[Max-Connection = 2048]</strong>
|---|---</p>
<hr />
<h2 id="성능-테스트-진행-중-nginx-worker-connection-error">[성능 테스트 진행 중 Nginx Worker Connection Error]</h2>
<p>부하 테스트를 진행하던 중 Nginx 로그를 확인해보니 아래와 같은 에러 로그가 기록되고 있었습니다.</p>
<blockquote>
<p><strong>에러 로그</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/347b17d6-c2c1-4c3f-b367-96d94ccee05f/image.png" /></p>
<p>1000개의 사용자 요청이 일어났을 때 발생하는 이 에러 메시지는 Nginx에서 동시에 처리할 수 있는 연결 수(worker_connections)가 부족하다는 경고이며, 구체적으로 Nginx의 현재 설정으로는 768개의 연결을 처리할 수 있도록 되어 있는데, 더 많은 연결이 들어와서 이 한계를 초과하였기에 발생한 에러로 판단하였습니다.</p>
<p><strong>따라서 기존에 768개의 사용자 요청을 처리할 수 있게 설정된 값을 2048로 변경하였습니다.</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/ba8e30e2-6cfb-4f0f-8cde-cca4feab0164/image.png" /> <strong>worker_connections 설정 변경</strong>| <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/a8780649-0263-459e-8e2b-da4bbfc7fbfc/image.png" /> <strong>병목 현상 발생 지점</strong>
|---|---</p>
<p>위의 그림과 같이 병목 현상을 최대한 줄이는 방식으로 구성하기 위해 <code>WorkerConnection</code> ≥ <code>Max-Connection</code>로 설정을 하는 것이 합리적이라고 생각했습니다. 따라서 <code>Max-Connection</code>의 수와 같이 2048개의 동시 처리 커넥션 수를 가져가기로 결정하였습니다.</p>
<hr />
<h2 id="tomcat-accept-count">[Tomcat-Accept-Count]</h2>
<p>Max-Connections 이상의 연결 시도가 있을 시 요청 대기열 큐에 저장되는데, 요청 대기열 큐의 사이즈인 Accept-Count 보다도 연결 시도가 많아지면 연결을 거부하게 됩니다.</p>
<p>Max-Connections도 충분히 작은 값으로 설정했기 때문에 수정하지 않기로(100개) 결정했습니다.</p>
<hr />
<h2 id="tomcat-max-thread">[Tomcat-Max-Thread]</h2>
<p><strong>Max-Thread</strong>의 크기가 크면 <code>Context Switching</code> 비용이 많이 발생하므로,</p>
<p>값을 줄이는 방향으로 테스트를 수행하였을 때 30일 때의 TPS가 가장 높게 측정되었습니다.</p>
<p><strong>Max-Thread</strong>의 수가 감소함에 따라 동일한 테스트 구간에서 TPS가 상승한 모습을 볼 수 있습니다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/6b4ad72f-c453-4e30-a83d-8f0582419f37/image.png" /> | <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/056a0fe9-162d-4bc0-8cdf-b92a751cb2aa/image.png" /> | <img alt="" src="https://velog.velcdn.com/images/ho-tea/post/5b18ec8d-1bb6-43f5-b49e-0424038749db/image.png" />
|---|---|---</p>
<hr />
<h2 id="tomcat-min-spare">[Tomcat-Min-Spare]</h2>
<p>항상 있는 최소한의 스레드 수로 너무 낮게 설정하면, 커넥션이 즉시 사용 가능한 상태가 아니어서 추가 요청 시 지연이 발생할 수 있고, 너무 높게 설정하면 불필요한 커넥션 유지로 자원 낭비가 발생할 수 있다고 판단되었습니다.</p>
<p>쓰레드 개수 10개 혹은 20개가 메모리 부하에 큰 영향을 미칠지, 성능에 큰 영향을 줄지 고민되어 값을 변경하면서 테스트를 해보았지만 큰 영향을 끼치지 않는다고 판단하여 수정하지 않기로(10개) 결정했습니다.</p>
<hr />
<h2 id="hikaricpmaximum-pool-size--hikaricpminimum-idle">[HikariCP.maximum-pool-size] &amp; [HikariCP.minimum-idle]</h2>
<p><strong>maximum-pool-size 와 minimum-idle을 동일하게가져가는 것이 권장사항</strong>이며, minimum-idle을 pool size보다 작게 가져갈 시 <strong>Connection을 생성하고 삭제하는 비용</strong>이 발생하므로, 동일하게 가져가는 방식을 선택하였습니다. </p>
<p><strong>maximum-pool-size</strong>을 기존 10보다 더 적게(5개) 혹은 더 많게(15개) 구성하였을 때, TPS와 성공응답 비율의 차이가 크지 않았다고 판단하여 <code>Default</code> 값 10으로 유지하였습니다.</p>
<p><a href="https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing">[🔗 HikariCP 공식 문서 참고]</a></p>
<hr />
<h2 id="hikaricpconnection-timeout">[HikariCP.connection-timeout]</h2>
<p>HikariCP의 <strong>connection-timeout</strong>이 클라이언트 측으로부터의 <strong>요청 timeout</strong>보다 짧아야 하기에 8초로 설정하였습니다.</p>
<p><strong>사용자가 10초 이상 기다리는 행동을 취하는 것 보다, 10초 내에 Connection이 맺어지지 않으면 빠른 실패를 한 후 재시도하게끔 구성하는 것이 사용자 경험 측면에서 이점이 있다고 생각되었습니다.</strong></p>
<p>따라서 클라이언트 측 timeout은 10초로 설정하게 되었고, 만약 HikariCP의 connection-timeout이 클라이언트 측보다 길게 된다면 서버에서 디비로 조회를 완료한 후 반환하려 할 때, 응답을 돌려주지 못하는 문제가 발생할 수 있기에 <strong>HikariCP의 connection-timeout은 클라이언트 측보다 짧게 가져간 8초로 설정하였습니다.</strong></p>
<hr />
<h1 id="tps-개선-결과">[TPS 개선 결과]</h1>
<blockquote>
<p><strong>TPS <code>30.6</code> → <code>44.3</code>으로 약 44%개선</strong></p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/eea702ca-5590-4bd8-9fb1-322aeba54f12/image.png" /></p>
<p>이 과정은 시스템 리소스를 효율적으로 활용하고 병목 지점을 최소화하여 애플리케이션의 처리 성능과 응답 속도를 크게 향상시킬 수 있는 중요한 작업이라고 생각됩니다. 서비스의 특성, 성격에 맞춰 애플리케이션을 튜닝하며 최적의 값을 찾아가는 과정은 서비스에 대한 깊은 이해를 쌓는 것에 큰 도움이 되었습니다.</p>