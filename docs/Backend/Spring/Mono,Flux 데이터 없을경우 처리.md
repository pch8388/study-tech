# Webflux 데이터가 없을 경우
- MVC 에서 특정 대상을 조회했는 데, 없을 경우에는 Optional 의 orElseThrow() 와 같은 메서드를 사용하여 예외를 발생시키는 방법을 사용하였었다
- Webflux 에서 Mono 나 Flux 사용시 데이터가 없다면 switchIfEmtpy 를 사용할 수 있다

```java
return this.postRepository.findById(postId)
  .flatMap(post -> {
    post.updateContent(updatePostDto.getUpdateContent());
    return this.postRepository.save(post);
  })
  .map(ResponsePostDto::convertFromEntity)
  .switchIfEmpty(Mono.error(new IllegalArgumentException("잘못된 post id")));
```

- switchIfEmpty 는 Mono(혹은 Flux) 가 비어있을 경우 대체할 Mono 를 정의할 수 있게 해준다 => 즉 Mono.error 를 반환하도록 하면 예외처리를 할 수 있음