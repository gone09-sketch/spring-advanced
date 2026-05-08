# Spring Advanced

## ✅ 구현 내용

### Lv 0. 프로젝트 세팅 - 에러 분석

**문제 상황**

애플리케이션 실행 시 `Application run failed` 에러 발생

**원인**

`application.properties`에 `jwt.secret.key` 값이 없어서 연쇄 빈 생성 실패

```
jwt.secret.key 없음
→ JwtUtil 빈 생성 실패
→ FilterConfig 빈 생성 실패
→ Tomcat 실행 실패
→ 애플리케이션 종료
```

**해결**

`application.properties`에 `jwt.secret.key` 값 추가

---

### Lv 1. ArgumentResolver

**문제 상황**

`AuthUserArgumentResolver`가 존재하지만 Spring에 등록되어 있지 않아 동작하지 않음

**해결**

`AuthUserWebMvcConfig` 클래스를 생성하여 `AuthUserArgumentResolver`를 Spring에 등록

---

### Lv 2. 코드 개선

**2-1. Early Return**

`signup()`에서 이메일 중복 체크를 앞으로 이동하여, 이미 존재하는 이메일인 경우 불필요한 `encode()` 호출 방지

```
기존: 암호화 → userRole 설정 → 이메일 검증
수정: 이메일 검증 → 암호화 → userRole 설정
```

**2-2. 불필요한 if-else 제거**

`WeatherClient`의 `getTodayWeather()`에서 첫 번째 `if`에서 예외를 던지면 메서드가 종료되므로 불필요한 `else` 블록 제거

**2-3. Validation**

`UserService`의 `changePassword()`에서 처리하던 비밀번호 유효성 검사 로직을 `UserChangePasswordRequest` DTO로 이동하여 `@Pattern` 어노테이션으로 대체

---

### Lv 3. N+1 문제

`TodoRepository`의 JPQL `fetch join` 쿼리를 `@EntityGraph` 기반 구현으로 변경

---

### Lv 4. 테스트 코드

**4-1. PasswordEncoderTest**

- 문제: `matches()` 파라미터 순서가 반대로 작성되어 있었음
- 해결: 실제 메서드 시그니처에 맞게 파라미터 순서 수정

**4-2. ManagerServiceTest - 메서드명 & 기대값 수정**

- 문제: 기대 메시지가 `"Manager not found"`로 잘못 작성되어 있었고, 메서드명에 `NPE`로 잘못 표기되어 있었음
- 해결: 실제 코드에서 던지는 메시지 `"Todo not found"`로 수정, 메서드명을 `InvalidRequestException`으로 수정

**4-3. CommentServiceTest - 예외 타입 수정**

- 문제: 테스트 코드에서 `ServerException`을 기대했지만 실제 코드는 `InvalidRequestException`을 던짐
- 해결: `assertThrows`의 예외 타입을 `InvalidRequestException`으로 수정

**4-4. ManagerServiceTest - 서비스 로직 수정**

- 문제: `todo.getUser()`가 null일 때 처리 로직이 없어 `InvalidRequestException` 대신 `NullPointerException` 발생
- 해결: 실제 서비스 코드에 null 체크 로직 추가
