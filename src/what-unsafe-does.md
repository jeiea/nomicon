# 언세이프 러스트로 할 수 있는 것

언세이프 러스트로만 가능한 게 있습니다.

* raw 포인터 참조하기
* `unsafe` 함수 호출 (C 함수, 컴파일러 내장 함수, 메모리 할당자 포함)
* `unsafe` 트레잇 구현
* 정적 변수 조작
* 공용체 필드 접근

이게 전부입니다. 이 기능들이 언세이프 러스트에 있는 이유는 잘못 사용하면 끔찍한
미정의 동작을 일으키기 때문입니다. 미정의 동작을 일으키는 건 컴파일러에게
모든 안 좋은 동작을 일으킬 권리를 주게 됩니다. 당신은 *절대* 미정의 동작을
일으키면 안 됩니다.

C와 달리 미정의 동작은 러스트에서 영역이 매우 제한되어 있습니다.
러스트는 언어 차원에서 다음을 막기 위해 신경쓰고 있습니다.

* (`*` 연산자를 사용한) 댕글링 포인터나 정렬되지 않은 포인터 참조 (아래 보기)
* [포인터 지시 규칙(pointer aliasing rule)][pointer aliasing rules] 위반
* 틀린 ABI를 사용한 함수 호출 또는 함수 되감기
* [데이터 레이스][race]
* 현재 실행 중인 스레드가 지원하지 않는 [타겟 피처][target features]로 컴파일 된 코드 실행
* (단독으로든 `enum`/`struct`/배열/터플같은 복합 타입의 필드로든) 말이 안 되는 값 사용
    * 0 또는 1이 아닌 `bool`
    * 틀린 판별 값을 사용한 `enum`
    * 널 `fn` 포인터
    * [0x0, 0xD7FF], [0xE000, 0x10FFFF] 범위를 벗어난 `char`
    * (어떤 값도 넣을 수 없는) `!`
    * [초기화되지 않은 메모리][uninitialized memory]에서 읽어온
      정수(`i*`/`u*`), 실수(`f*`) 또는 raw 포인터
    * 해제되었거나 미정렬이거나 틀린 값을 가리키는 레퍼런스/`Box`
    * 틀린 메타데이터를 가진 와이드 레퍼런스/`Box`/raw 포인터
        * 구현한 동적 트레잇의 vtable을 가리키지 않는 `dyn Trait` 메타데이터
        * 틀린 `usize` 길이를 가진 슬라이스 메타데이터
          (즉 초기화되지 않은 메모리에서 읽히면 안 됨)
    * UTF-8이 아닌 `str`
    * null인 `NonNull` 같은 위와 같은 무효한 값을 가진 타입. (무효한 값을
      요청하는 건 안정화되지 않은 기능이지만 `NonNull` 같은 몇몇 libstd
      타입은 사용하고 있습니다.)

값을 "생성"하는 건 값을 할당하거나 함수/기본 연산에 넘겨줄 때 또는
함수/기본 연산에서 넘겨받을 때 일어납니다.

레퍼런스/포인터는 그게 널을 가리키거나 가리킨 모든 범위가 같은 할당 영역에
들어가지 않을 때 "댕글링"이라고 합니다 (그러므로 모든 레퍼런스/포인터는 *어떤*
할당의 일부분이어야 합니다). 레퍼런스/포인터가 가리키는 바이트 범위는 포인터의
값과 포인터 타입의 크기로 결정됩니다. 만약 범위가 비어있다면 "댕글링"은
"non-null"과 같습니다. 슬라이스는 스스로의 모든 범위를 가리키기에 길이
메타데이터가 커지지 않는 게 중요합니다. (특히 할당과 관련되기에 슬라이스는
`isize::MAX` 바이트를 넘을 수 없습니다). 만약 어떤 이유로 범위를
다루기 힘들다면 raw 포인터를 쓰는 걸 고려해보길 바랍니다.

이걸로 끝입니다. 이게 러스트에 미정의 동작을 초래할 수 있는 모든 이유입니다.
물론, 불안전 함수와 트레잇이 미정의 동작을 피하기 위해 지켜야하는 다른
조건을 추가할 수 있습니다. 예를 들어, 할당자 API는 할당되지 않은 메모리를
해제하는 건 미정의 동작을 일으킨다고 선언하고 있습니다.

하지만, 이 조건들을 위반하는 건 위에 나온 다른 문제들을 덩달아 유발할 수 있습니다.
몇몇 추가 제약 조건은 코드를 최적화하기 위해 특별한 전제를 하는 컴파일러
내부 함수에서 파생하기도 합니다. 예를 들어, `Vec`과 `Box`는 사용하는 포인터가
항상 널이 아님을 요구하는 컴파일러 내부 함수를 씁니다.

러스트는 다른 애매한 작업에 관해선 상당히 관대합니다.
아래 작업들은 "안전"하다고 간주합니다.

* 교착상태
* [레이스 컨디션][race]
* 메모리 누수
* 정리자 호출 실패
* 정수 오버플로우
* 프로그램 중단
* 배포 환경의 데이터베이스 삭제

하지만 이런 짓을 하려는 프로그램은 *아마도* 잘못 되었을 겁니다.
러스트는 이런 행동을 최소화하는 여러 수단을 제공하지만, 이것들을
완전히 막는 것은 실용적이지 않습니다.

[pointer aliasing rules]: references.html
[uninitialized memory]: uninitialized.html
[race]: races.html
[target features]: ../reference/attributes/codegen.html#the-target_feature-attribute
