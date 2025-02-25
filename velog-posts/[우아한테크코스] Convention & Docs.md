<blockquote>
<p><strong>우아한테크코스 6기 최종 코딩테스트</strong>를 준비하면서 작성된 글의 일부로,
해당 글은 <strong>오로지 최종 코딩테스트</strong>에만 대비하기 위해 작성된 글입니다.</p>
</blockquote>
<p>아래의 우아한테크코스의 프리코스 과제를 수행해 오면서 정리한 내용들로 이루어져 있습니다.</p>
<ul>
<li><code>oncall</code> - 최종 코딩 테스트!</li>
<li><code>subway-path</code></li>
<li><code>pairmatching-precourse</code></li>
<li><code>bridge</code></li>
<li><code>baseball</code></li>
<li><code>menu</code></li>
<li><code>christmas</code></li>
<li><code>lotto</code></li>
<li><code>racingcar</code></li>
<li><code>vendingmachine</code></li>
<li><code>onboarding</code></li>
</ul>
<hr />
<h2 id="convention">Convention</h2>
<h3 id="angularjs-commit-convention">AngularJS commit convention</h3>
<pre><code>docs(README): 기능 목록 정리</code></pre><h3 id="google-java-style-guide-code-convention"><a href="https://github.com/woowacourse/woowacourse-docs/tree/main/styleguide/java">Google Java Style Guide (Code convention)</a></h3>
<ul>
<li><a href="https://vince-kim.tistory.com/28">Intellij Google JAVA Style Guide 설정</a></li>
<li><a href="https://velog.io/@pgmjun/IntelliJ-%EC%BD%94%EB%93%9C-%EC%8A%A4%ED%83%80%EC%9D%BC%EC%9D%84-%EC%84%A4%EC%A0%95%ED%95%B4%EB%B3%B4%EC%9E%90-feat.%EC%9A%B0%ED%85%8C%EC%BD%94">Intellij WootecoStyle 설정</a></li>
</ul>
<hr />
<h2 id="docs">Docs</h2>
<h3 id="기능-요구-사항-프로그래밍-요구-사항-과제-진행-요구-사항-을-만족하기-위해-노력한다">기능 요구 사항, 프로그래밍 요구 사항, 과제 진행 요구 사항 을 만족하기 위해 노력한다.</h3>
<p>특히 기능 요구 사항을 읽어가면서, 처음에는 <code>README</code>에 아래의 내용을 작성해 나아간다. 기능 요구 사항을 읽어도 눈에 잘 안들어 올 수 있다.
따라서 <code>README</code>에 중요하다고 생각되는 부분 먼저 작성하고,</p>
<ul>
<li><strong>Q. 해당 도메인이 이런 책임을 가져도 되는지 애매하다면?
A. 우선 그 도메인에 해당 기능을 구현하고 추후에 분리하자!</strong></li>
</ul>
<p><strong>해당 도메인을 구현하면서 분리가 가능하다고 생각되면 분리를 진행하자.</strong></p>
<blockquote>
<p><strong>초기 README</strong></p>
</blockquote>
<ul>
<li>전체적인 게임 규칙</li>
<li>전체적인 게임 흐름</li>
<li>Domain</li>
<li>Input/Output</li>
<li>Controller</li>
</ul>
<h3 id="💡-도메인과-입력이-중복되는-부분이-반드시-존재한다---중복-제거-❌">💡 도메인과 입력이 중복되는 부분이 반드시 존재한다. - 중복 제거 ❌</h3>
<p><strong>다리</strong> : <code>다리의 길이는 3 이상 20 이하로 만들어져야 한다.</code></p>
<p><strong>입력</strong> : <code>자동으로 생성할 다리 길이를 입력 받는다. 3 이상 20 이하의 숫자를 입력할 수 있다</code></p>
<p>이와 같은 부분은 도메인의 <strong>비즈니스 로직 예외 검증 부분</strong>과, 
입력의 <strong>단순 입력 예외 검증 부분</strong>을 구분하는 용도로 중복을 제거하지 말고, 그대로 유지한다.</p>
<h3 id="💡-중복되는-기능-요구사항들을-합치자---중복-제거-⭕️">💡 중복되는 기능 요구사항들을 합치자. - 중복 제거 ⭕️</h3>
<p>하나의 도메인<code>(ex: 입력, 출력, 다리 등)</code>에서 중복되는 요구사항들이 존재한다면 다른 도메인으로 전파 시키지 말고 그 도메인 내에서 중복을 제거하자.</p>
<h3 id="💡-단순-생성자만-존재해도-readme에-추가하자">💡 단순 생성자만 존재해도 README에 추가하자</h3>
<p>단순히 값을 표현하는 <code>Enum</code>객체에서 <code>생성자</code>와 <code>Getter</code>만 존재하더라도 기능 목록에 작성해 주고, 필드<code>(생성자만 있는 클래스, 열거형)</code>만 가지고 있는 경우 
<code>README</code>작성 시 <code>- [x]</code>로 <strong>체크박스화</strong> 하지말고 작성하자!
정의할 수 있다 보다는 <code>구성된다</code> 가 더 눈에 잘들어오는 것 같다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/e227c29d-299b-4811-affc-122cb7e9fa51/image.png" /></p>
<blockquote>
<p><strong>최종 README</strong></p>
</blockquote>
<ul>
<li><strong>Domain</strong></li>
<li><strong>Input/Output</strong></li>
<li><strong>Controller</strong></li>
</ul>