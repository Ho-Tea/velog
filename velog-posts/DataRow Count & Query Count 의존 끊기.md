<blockquote>
<p><strong>본 프로젝트는 Spring Boot와 MySQL을 활용한 기록 프로젝트 입니다.</strong></p>
</blockquote>
<p>해당 글은 기록 프로젝트 진행 중, 특정 레코드 삭제 시 JpaRepository와 Cascade로 인해 불필요한 SELECT 및 개별 DELETE 쿼리가 다량 발생하는 문제를 해결하고자 선택한 기술적 접근 방식에 대해 설명합니다.</p>
<hr />
<h1 id="요약">[요약]</h1>
<p>Memory 삭제 API 호출 시, 관련 Moment와 그에 속한 Comment, MomentImage 등 연관 엔티티 삭제 과정에서 약 200개의 쿼리가 발생하는 문제를, JPQL 기반의 <strong><code>Bulk Delete</code></strong>를 활용해 조건에 맞는 데이터만 일괄 삭제함으로써 총 쿼리 수를 5개로 줄여 성능 최적화를 달성하였습니다.</p>
<hr />
<h1 id="배경-지식">[배경 지식]</h1>
<pre><code class="language-java">// Moment.java

@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = &quot;memory_id&quot;, nullable = false)
private Memory memory;
@Embedded
private MomentImages momentImages = new MomentImages();
@OneToMany(mappedBy = &quot;moment&quot;, orphanRemoval = true, cascade = CascadeType.ALL)
private List&lt;Comment&gt; comments = new ArrayList&lt;&gt;();

// MomentImages.java
@OneToMany(mappedBy = &quot;moment&quot;, orphanRemoval = true, cascade = CascadeType.ALL)
private List&lt;MomentImage&gt; images = new ArrayList&lt;&gt;();</code></pre>
<p>Memory(부모 엔티티)가 N개의 Moment(자식 엔티티)를 가지고, Moment(부모 엔티티)가 N개의 Comment, MomentImage (자식 엔티티)를 가집니다. <strong>(양방향 연관관계 설정)</strong></p>
<p><img alt="" src="https://velog.velcdn.com/images/ho-tea/post/74100d76-e6ba-4c8f-a9f6-31722b93c581/image.png" /></p>
<h1 id="문제-상황">[문제 상황]</h1>
<p>특정 Memory(추억에 해당하는 도메인) 삭제 API 호출 시, 불필요한 <code>Select Query</code>와 <code>Delete Query</code>가 <code>Data row</code> 수만큼 나가는 문제상황이 발생했습니다.</p>
<pre><code class="language-java">// Memory.java

@Transactional
public void deleteMemory(long memoryId, Member member) {
    memoryRepository.findById(memoryId).ifPresent(memory -&gt; {
        validateOwner(memory, member);
        momentRepository.deleteAllByMemoryId(memoryId);
        memoryRepository.deleteById(memoryId);
    });
}</code></pre>
<ul>
<li>moment 별 comment, moment_image에 대해 <code>Select Qeury</code> 발생</li>
<li>조회된 모든 것들마다 개별적으로 <code>Delete Query</code> 발생</li>
</ul>
<hr />
<h2 id="문제-상황-예시">[문제 상황 예시]</h2>
<p><strong>moment가 10개, moment 당 comment가 10개, moment_image가 5개라고 한다면</strong></p>
<ul>
<li>특정 memory id를 가지는 moment 를 조회하는 쿼리 <code>1</code>개</li>
<li>특정 moment에서 모든 comment 조회하는 쿼리 <code>10</code>개<ul>
<li>comment 삭제 쿼리 10(moment 수) X 10(comment 수) = <code>100</code>개</li>
</ul>
</li>
<li>특정 moment에서 모든  moment_image 조회하는 쿼리 <code>10</code>개<ul>
<li>moment_image 삭제 쿼리 10(moment 수) X 5(moment_image 수) = <code>50</code>개</li>
</ul>
</li>
<li>moment 삭제 쿼리 <code>10</code>개</li>
<li>memory_member 삭제 쿼리 <code>1</code>개</li>
<li>memory 삭제 쿼리 <code>1</code>개</li>
</ul>
<p><code>= 약 200개의 쿼리가 나가게 됩니다.</code></p>
<hr />
<h2 id="문제-상황-분석">[문제 상황 분석]</h2>
<pre><code class="language-java">// Memory.java

@Transactional
public void deleteMemory(long memoryId, Member member) {
    memoryRepository.findById(memoryId).ifPresent(memory -&gt; {
        validateOwner(memory, member);
        **momentRepository.deleteAllByMemoryId(memoryId);**
        **memoryRepository.deleteById(memoryId);**
    });
}

// SimpleJpaRepository.java

@Transactional
**public void deleteById(ID id) {**
    Assert.notNull(id, &quot;The given id must not be null&quot;);
    this.findById(id).ifPresent(this::delete);
}

...

@Transactional
**public void deleteAllById(Iterable&lt;? extends ID&gt; ids) {**
    Assert.notNull(ids, &quot;Ids must not be null&quot;);
    Iterator var3 = ids.iterator();

    while(var3.hasNext()) {
        ID id = (Object)var3.next();
        this.deleteById(id);
    }
}

...

**public Optional&lt;T&gt; findById(ID id) {**
    Assert.notNull(id, &quot;The given id must not be null&quot;);
    Class&lt;T&gt; domainType = this.getDomainClass();
    if (this.metadata == null) {
        return Optional.ofNullable(this.entityManager.find(domainType, id));
    } else {
        LockModeType type = this.metadata.getLockModeType();
        Map&lt;String, Object&gt; hints = this.getHints();
        return Optional.ofNullable(type == null ? this.entityManager.find(domainType, id, hints) : this.entityManager.find(domainType, id, type, hints));
    }
}
...
@Transactional
**public void delete(T entity) {**
    Assert.notNull(entity, &quot;Entity must not be null&quot;);
    if (!this.entityInformation.isNew(entity)) {
        Class&lt;?&gt; type = ProxyUtils.getUserClass(entity);
        T existing = this.entityManager.find(type, this.entityInformation.getId(entity));
        if (existing != null) {
            this.entityManager.remove(this.entityManager.contains(entity) ? entity : this.entityManager.merge(entity));
        }
    }
}</code></pre>
<ol>
<li><strong>deleteAllById(Iterable&lt;? extends ID&gt; ids)</strong> 호출 시 <strong>ids 를 순회하면서 deleteById(ID id)</strong> 를 호출</li>
<li><strong>deleteById(ID id)</strong>는 내부적으로 <strong>em.find()</strong> 를 통해 지우려는 <strong><code>entity 를 영속성 컨텍스트에 등록</code></strong></li>
<li>이후 <strong>em.remove()</strong> 를 호출. <ol>
<li>여기서 <code>Cascade</code> 가 걸려있는 엔티티도 삭제하기 위해 영속성 컨텍스트에 등록하는 과정에서 <code>select</code> 쿼리가 추가적으로 발생.</li>
</ol>
</li>
</ol>
<p><strong><code>JpaRepository</code> 가 제공하는 <code>delete</code> 를 활용하게 되면 예상치 못한 <code>select</code> 쿼리가 발생한다는 것을 알 수 있었습니다.</strong></p>
<hr />
<h1 id="해결-방안">[해결 방안]</h1>
<h2 id="✅-해결-방안-1---직접적인-delete-쿼리-사용으로-일괄-삭제bulk-delete">✅ [해결 방안 1 - 직접적인 DELETE 쿼리 사용으로 일괄 삭제(Bulk Delete)]</h2>
<ul>
<li>성능 최적화를 위해 직접적인 DELETE 쿼리를 작성</li>
<li><strong>JPQL을 사용하여 조회 없이 한 번의 쿼리로 삭제</strong></li>
<li><strong>in절을 활용해 여러 조건에 해당하는 데이터를 한 번에 삭제할 수 있도록 구성</strong></li>
</ul>
<pre><code class="language-java">@Modifying
@Query(&quot;DELETE FROM Comment c WHERE c.moment.id IN :momentIds&quot;)
void deleteAllByMomentIdInBulk(@Param(&quot;momentIds&quot;) List&lt;Long&gt; momentIds);</code></pre>
<hr />
<h2 id="❌-해결-방안-2---spring-data-jpa가-제공하는-deleteallinbatch-메서드-사용">❌ [해결 방안 2 - Spring Data JPA가 제공하는 deleteAllInBatch 메서드 사용]</h2>
<ul>
<li><code>deleteAllInBatch()</code> 메서드는 엔티티 리스트 전체를 한 번의 SQL 문으로 삭제</li>
</ul>
<pre><code class="language-java">momentRepository.deleteAllInBatch();</code></pre>
<hr />
<h2 id="해결방안-1bulk-delete-선택-이유">[해결방안 1(Bulk Delete) 선택 이유]</h2>
<h3 id="조건-기반-삭제의-필요성">[조건 기반 삭제의 필요성]</h3>
<p><code>Bulk delete</code>는 특정 조건에 맞는 데이터를 선택적으로 삭제할 수 있는 장점이 있습니다. 만약 특정 조건에 맞는 일부 데이터만 삭제하고자 할 경우, <code>deleteAllInBatch()</code>는 사용할 수 없고 <code>Bulk Delete</code>를 사용해야 합니다. 물론 <code>deleteAllByIdInBatch()</code>와 같이 특정 식별자 목록을 기반으로 일괄 삭제하는 메서드를 활용할 수도 있지만, 이는 자신 테이블의 ID에 해당하는 데이터만 삭제하기에 사용할 수 없었습니다.</p>
<p>따라서 아래와 같이 <strong><code>BulkDelete</code></strong>를 활용하는 코드로 개선되었습니다.</p>
<blockquote>
<p><strong>Memory → Category, Moment → Staccato 로 도메인 명 변경이 이루어졌습니다.</strong></p>
</blockquote>
<pre><code class="language-java">@Transactional
public void deleteCategory(long categoryId, Member member) {
    categoryRepository.findById(categoryId).ifPresent(category -&gt; {
        validateOwner(category, member);
        deleteAllRelatedCategory(categoryId);
        categoryRepository.deleteById(categoryId);
    });
}

private void deleteAllRelatedCategory(long categoryId) {
    List&lt;Long&gt; staccatoIds = staccatoRepository.findAllByCategoryId(categoryId)
            .stream()
            .map(Staccato::getId)
            .toList();
    staccatoImageRepository.deleteAllByStaccatoIdInBulk(staccatoIds);
    commentRepository.deleteAllByStaccatoIdInBulk(staccatoIds);
    staccatoRepository.deleteAllByCategoryIdInBulk(categoryId);
    categoryMemberRepository.deleteAllByCategoryIdInBulk(categoryId);
}</code></pre>
<hr />
<h2 id="개선된-문제-상황">[개선된 문제 상황]</h2>
<h3 id="문제-상황-예시에서-발생했던-200개의-쿼리가-5개로-줄일-수-있었습니다">[문제 상황 예시에서 발생했던 200개의 쿼리가 5개로 줄일 수 있었습니다.]</h3>
<p><strong>Category가 10개, Staccato 당 comment가 10개, staccato_image가 5개라고 한다면</strong></p>
<ul>
<li>특정 category_id를 가지는 staccato_ids 를 조회하는 쿼리 <code>1</code>개</li>
<li>staccato_ids에 포함되는 staccato_id를 가지는 모든 comment 삭제하는 쿼리 <code>1</code>개</li>
<li>staccato_ids에 포함되는 staccato_id를 가지는 모든 staccato_image 삭제하는 쿼리 <code>1</code>개</li>
<li>category_member 삭제 쿼리 <code>1</code>개</li>
<li>category 삭제 쿼리 <code>1</code>개</li>
</ul>
<h1 id="결론"><strong>[결론]</strong></h1>
<p>연관 엔티티 삭제 시 JpaRepository와 Cascade 설정으로 인해 불필요하게 대량의 SELECT 및 DELETE 쿼리가 발생하는 문제를 확인하였습니다. 이를 해결하기 위해, JPQL 기반의 Bulk Delete 기법을 도입하여 조건에 맞는 데이터를 일괄 삭제함으로써 쿼리 발생 횟수를 약 200개에서 5개로 줄일 수 있었습니다.</p>
<p>이와 같은 접근 방식은 단순히 쿼리 수를 줄여 데이터베이스 부하를 경감시키는 것을 넘어, 시스템의 성능과 응답 속도를 향상시키는 효과를 가져올 수 있다고 생각합니다.</p>