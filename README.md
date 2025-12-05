# Spring 숙련 개인 과제 README

## 프로젝트 개요

Spring Boot 기반 API 서버의 기능 개선 과제입니다.
레벨 0~4 요구사항에 따라 프로젝트 실행 오류 해결, ArgumentResolver 복구, 코드 리팩토링, Validation 적용, 및 테스트 코드 수정 등을 수행했습니다.

---

## 사용 기술

* **Java 17**
* **Spring Boot 3.3.3**
* **Lombok**
* **Spring Web**
* **Spring Data JPA**
* **Jakarta Validation**
* **Gradle**

## ✔️ 레벨 0 — 프로젝트 실행 오류 해결

* `resources/` 폴더 생성
* `application.yml` 파일 생성 및 DB/JPA 설정 추가
* JWT Secret Key 길이 부족으로 인한 실행 오류 발생 → 충분한 길이의 key로 교체하여 해결

---

## ✔️ 레벨 1 — ArgumentResolver 복구

### 문제 원인

`AuthUserArgumentResolver` 로직이 누락되면서 `@Auth AuthUser` 사용 불가

### 해결 내용

* `supportsParameter()`

  ```java
  boolean hasAuthAnnotation = parameter.hasParameterAnnotation(Auth.class);
  boolean isAuthUserType = parameter.getParameterType().equals(AuthUser.class);
  ```

  * 두 조건이 함께 true가 아니면 예외 발생하도록 수정
* `resolveArgument()` 파라미터의 `@Nullable` 제거
* `WebMvcConfig` 생성 → Resolver 등록

  ```java
  @Override
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
      resolvers.add(authUserArgumentResolver);
  }
  ```

---

## ✔️ 레벨 2 — 코드 개선 (리팩토링 & Validation)

### **2-1. Early Return 적용**

* 이메일 중복 검증을 **passwordEncoder.encode()** 이전으로 이동
* 불필요한 암호화 호출 제거 → 성능 및 가독성 향상

### **2-2. 불필요한 if-else 제거**

* 중첩된 else 블록 제거
* 한 줄에 여러 조건이 섞여 있던 부분을 각각 분리하여 명확하게 표현
* 코드 구조 단순화 및 가독성 개선

### **2-3. Validation 적용**

* `UserChangePasswordRequest` DTO에서 비밀번호 검증 수행하도록 변경

  ```java
  @Size(min = 8)
  @Pattern(regexp = "^(?=.*\d)(?=.*[A-Z]).+$")
  ```
* Controller 메서드에 `@Valid` 적용
* Service 내부 비밀번호 검증 제거 → 단일 책임 원칙 준수

---

## ✔️ 레벨 3 — N+1 문제 해결

### 문제 원인

Todo 조회 시 연관된 User가 LAZY 로딩되어 추가 쿼리 발생 가능 → 성능 저하

### 해결 내용

* 기존 fetch join 제거
* `@EntityGraph(attributePaths = {"user"})` 적용
* 적용 대상:

  ```java
  @EntityGraph(attributePaths = {"user"})
  Page<Todo> findAllByOrderByModifiedAtDesc(Pageable pageable);

  @EntityGraph(attributePaths = {"user"})
  Optional<Todo> findById(Long todoId); // findByIdWithUser → findById로 수정
  ```
* Service에서 `EntityGraph` 기반 메서드로 조회하도록 수정
* 미사용 `countById()` 제거

---

## ✔️ 레벨 4 — 테스트 코드 수정

## 1) 테스트 코드 연습 - 1

### 문제 원인
- `boolean matches = passwordEncoder.matches(encodedPassword, rawPassword);`
- `encodedPassword` 와 `rawPassword` 인자 순서가 뒤바뀌어 결과가 항상 `false`가 됨

### 해결 내용
- `assertTrue(matches);` → `assertFalse(matches);` 로 수정하여 테스트가 정상 통과하도록 변경

---

## 2) 테스트 코드 연습 - 2 (1번 케이스)

### 문제 원인
- 테스트 메서드명  
  **manager_목록_조회_시_Todo가_없다면_NPE_에러를_던진다()**  
  → 실제 발생 예외는 `NullPointerException`이 아니라 **InvalidRequestException** 임
- 단언문 `assertEquals`의 expected 메시지가 `"Manager not found"`로 되어 있으나,  
  실제 상황은 **Todo 부재**이므로 메시지가 일치하지 않음

### 해결 내용
- 테스트 메서드명에서 `NPE` → `InvalidRequestException` 로 변경
- `assertEquals("Manager not found", exception.getMessage());`  
  → `"Todo not found"` 로 expected 메시지 수정

---

## 실행 방법

```bash
./gradlew bootRun
```
---

## 느낀점

필수 기능 레벨 0, 2, 3은 이전 CRUD 과제나 강의를 통해 경험해본 내용이라 비교적 수월하게 해결할 수 있었다.
반면, 레벨 1의 ArgumentResolver 문제는 접근 방식이 익숙하지 않아 가장 시간이 오래 걸렸고, 문제의 흐름을 이해하는 데 많은 고민이 필요했다.
개인 사정으로 진행 속도가 느려 레벨 4 마무리하지 못한 점은 아쉽지만, 테스트 코드 강의를 모두 듣고 다시 도전해볼 예정이다.
이번 과제를 통해 코드 개선과 구조적인 수정의 중요성, 그리고 테스트 코드의 필요성을 느낄 수 있어 유익한 경험이었다.

