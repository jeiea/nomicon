# 언세이프 러스트로 작업하기

러스트는 우리가 언세이프 러스트를 범위 제한적이고 명확한 방식으로 다룰 수 있는
수단을 제공합니다. 불행히도 현실은 그것보단 복잡합니다. 아래 함수를 봅시다.

```rust
fn index(idx: usize, arr: &[u8]) -> Option<u8> {
    if idx < arr.len() {
        unsafe {
            Some(*arr.get_unchecked(idx))
        }
    } else {
        None
    }
}
```

이 함수는 안전하고 올바릅니다. 인덱스를 확인하고 유효 범위 안이면 배열에서
확인하지 않고 꺼내옵니다. 우리는 그런 안전하지 않아도 올바르게 구현한
함수를 *건전(sound)*하다고 말하고, 건전하다는 건 세이프 코드가
그 언세이프 함수를 통해 미정의 동작을 일으킬 수 없다는 걸 의미합니다.
(이건 세이프 러스트의 기본적인 속성입니다)

하지만 이 간단한 함수조차도, 불안전 블록 범위가 의문입니다.
`<`를 `<=`로 바꿔보면 어떨까요.

```rust
fn index(idx: usize, arr: &[u8]) -> Option<u8> {
    if idx <= arr.len() {
        unsafe {
            Some(*arr.get_unchecked(idx))
        }
    } else {
        None
    }
}
```

우린 *세이프 코드만 수정했을 뿐*인데도 프로그램은 이제 *건전하지 않고*,
세이프 러스트는 미정의 동작을 일으킬 수 있습니다. 지역적이지 않다는 건
안전성의 근본적인 문제입니다. 언세이프 연산의 건전함은 다른 "세이프"
연산의 상태에 의존합니다.

안정성은 언세이프 코드에서 당신이 다른 종류의 낭패를 걱정하지 않아도 된단
정에선 격리적입니다. 확인하지 않은 인덱스로 슬라이스를 만든다고 슬라이스가
널이 되거나 초기화되지 않은 메모리를 포함하는 걸 걱정하지는 않아도 된다는
얘깁니다. 근본적인 무언가가 변하진 않습니다. 하지만 안전성은 프로그램이
본질적으로 상태 의존적이고 직접 짠 언세이프 연산이 다른 상태에 의존적일 수
있단 점에서 격리적이진 않습니다.

이 비지역성은 우리가 지속적인 실제 상태와 통합할 때 더 심각해집니다.
다음 `Vec`의 간단한 구현을 봅시다.

```rust
use std::ptr;

// 참고: 이 정의는 허술합니다. Vec을 구현하는 챕터를 보세요.
pub struct Vec<T> {
    ptr: *mut T,
    len: usize,
    cap: usize,
}

// 이 구현은 크기가 0인 타입을 제대로 다루지 못합니다.
// Vec을 구현하는 챕터를 보세요.
impl<T> Vec<T> {
    pub fn push(&mut self, elem: T) {
        if self.len == self.cap {
            // 이 예제에선 중요하지 않습니다
            self.reallocate();
        }
        unsafe {
            ptr::write(self.ptr.offset(self.len as isize), elem);
            self.len += 1;
        }
    }
    # fn reallocate(&mut self) { }
}

# fn main() {}
```

위 코드는 문제가 없다고 확신하기 쉬울만큼 충분히 단순합니다. 이제 다음 메소드를
추가한다고 해봅시다.

```rust,ignore
fn make_room(&mut self) {
    // 적재 용량 증가
    self.cap += 1;
}
```

이 코드는 100% 세이프 러스트지만 동시에 절대 건전하지 못합니다. 적재 용량을
바꾸는 건 (`cap`이 Vec의 할당 공간을 나타내기에) Vec의 불변 속성을 건드립니다.
이건 Vec의 나머지 부분이 방어할 수 있는 행동이 아닙니다. Vec은 `cap` 필드가
잘못됐는지 확인할 방법이 없기 때문에 반드시 믿어야 *합니다*.

Vec이 구조체 필드의 불변 속성에 의지하기 때문에, 이 `unsafe` 코드는 전체
함수를 오염시키는 걸 넘어서 전체 *모듈*을 오염시킵니다.
일반적으로 언세이프 코드의 범위를 제한하는 확실한 수단은 모듈 경계에서
가시성을 조절하는 겁니다.

하지만 이건 *완벽히* 동작합니다. `make_room`을 공개하지 않았기 때문에 `make_room`
이 있다고 해서 Vec의 건전함이 사라지진 않습니다. `make_room`을 정의한 모듈만이
`make_room`을 호출할 수 있을 겁니다. 또한 `make_room`은 Vec의 프라이빗 필드에
접근하기에 Vec과 같은 모듈에서만 짤 수 있을 겁니다.

이런 이유로 복잡한 불변 조건에 의존하는 완벽히 안전한 추상화를 짤 수 있습니다.
이건 세이프 러스트와 언세이프 러스트 사이의 가장 *중요한* 관계입니다.

우린 이미 언세이프 코드가 *일부* 세이프 코드를 믿을 순 있지만, *제네릭*
안전 코드는 믿으면 안된단 걸 봤습니다. 비슷한 이유로 불안전 코드에게 가시성은
중요합니다. 가시성은 모든 세이프 코드를 믿는 것을 방지함으로써 우리가 신뢰하는
상태를 망치는 걸 예방합니다.

안전성 만세!

