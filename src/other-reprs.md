# 대체 레이아웃

러스트에서 다른 데이터 레이아웃 방식을 지정할 수 있습니다. [참고 문서][reference]
도 있습니다.




# repr(C)

이건 가장 중요한 `repr`입니다. C가 하는대로라는 무척 단순한 의도입니다.
순서, 크기, 필드 정렬이 C나 C++에서 기대했던 대로 짜일 것입니다.
C가 프로그래밍 언어의 사실상 표준이기에 FFI 경계 너머로 넘길 모든 타입은
반드시 `repr(C)`를 가져야 합니다. 구조체를 다른 타입으로 재해석하는 트릭을
제대로 사용하는 데도 필요합니다.

FFI 경계를 관리하기 위해 [rust-bindgen][]이나 [cbindgen][]을 쓸 것을 강권합니다.
러스트 팀은 이 프로젝트들이 견고하게, 현재와 미래에 타입 레이아웃과 repr에 대한
전제를 호환하도록 밀접하게 작업하고 있습니다.

하지만 `repr(C)`로 러스트의 더 외부의 데이터 레이아웃 기능을 다루는 데엔
명심할 것이 있습니다. FFI와 레이아웃 제어를 위해서라는 목적의 양면성 때문에,
`repr(C)`는 FFI 경계 너머로 넘기면 말이 안 되거나 문제가 생기는 타입에
적용될 수 있습니다.

* ZST는 repr(C)를 적용하더라도 크기를 가지지 않습니다. 비록 C의
  표준 동작과 일치하지 않고, C++의 빈 타입이 1 바이트를 차지해야
  한다는 것에 모순되더라도 말입니다.

* DST 포인터 (와이드 포인터), 터플은 C에 있는 개념이
  아니기에 FFI에서 안전할 수 없습니다.

* 필드를 가진 열거형 또한 C나 C++에 없지만, 타당한 타입 변환이
  [정의되었습니다][really-tagged]

* `T`가 [FFI 안전한 널이 불가능한 포인터 타입](ffi.html#the-nullable-pointer-optimization)이면,
  `Option<T>`는 `T`와 같은 레이아웃과 ABI를 가지고 그 결과 FFI 안전합니다.
  이 글이 쓰여지는 시점 기준으로 이 규칙은 널이 될 수 없는 `&`, `&mut`과
  함수 포인터에 적용됩니다.

* `repr(C)`에서 터플 구조체는 일반 구조체와 같습니다.
  유일한 차이점은 필드에 이름이 없다는 겁니다.

* `repr(C)`는 아래에서 소개할 `repr(u*)`의 한 종류와 필드없는 열거형 측면에서
  동일합니다. 선택하는 크기는 대상 플랫폼 C ABI의 기본 열거형 크기가 됩니다.
  C의 열거형 표현은 구현에 따라 다르기 때문에 "최선의 추측"을 할 뿐입니다.
  특히, 목표로 한 C코드가 특별한 플래그로 컴파일 되었을 때 부정확해질 수
  있습니다.

* C나 C++에선 가능했지만 `repr(C)`나 `repr(u*)` 지정을 한 필드없는
  열거형은 정의하지 않은 정수 값으로 설정할 수 없습니다. 안전하지 않은
  코드로 정의하지 않은 정수 값을 사용해 열거형 인스턴스를 만드는 것은
  미정의 동작입니다. (이 경우 전수 조사 매칭을 여전히 작성할
  수 있고 컴파일도 정상적으로 됩니다.)


# repr(transparent)

이 어트리뷰트는 크기가 0이 아닌 필드 하나만을 가진 구조체에만 쓸 수 있습니다
(크기가 0인 필드가 추가로 있어도 됩니다). 효과는 구조체 전체 레이아웃과 ABI가
그 필드 하나와 동일해지는 것입니다.

이걸 쓰는 이유는 단일 필드와 구조체 간 변환을 가능하게 하기 위함입니다.
한 예로  [`UnsafeCell`]은 그 타입이 감싼 타입으로 바꿀 수 있습니다.


또한 해당 구조체를 필드 안의 타입을 요구하는 FFI 너머로 넘기는 것도
정상 동작이 보장됩니다. 특히 `struct Foo(f32)`가 `f32`와 같은 ABI를
무조건 가지게 할 때 필요합니다.

자세한 내용은 [RFC][rfc-transparent]를 참고하세요.



# repr(u*), repr(i*)

이 어트리뷰트들은 필드 없는 열거형의 크기를 지정합니다. 열거형의 구분 태그가
지정한 수를 벗어나면, 컴파일 에러를 일으킵니다.
manually ask Rust to allow this by setting the overflowing element to explicitly
be 0. However Rust will not allow you to create an enum where two variants have
the same discriminant.

The term "fieldless enum" only means that the enum doesn't have data in any
of its variants. A fieldless enum without a `repr(u*)` or `repr(C)` is
still a Rust native type, and does not have a stable ABI representation.
Adding a `repr` causes it to be treated exactly like the specified
integer size for ABI purposes.

If the enum has fields, the effect is similar to the effect of `repr(C)`
in that there is a defined layout of the type. This makes it possible to
pass the enum to C code, or access the type's raw representation and directly
manipulate its tag and fields. See [the RFC][really-tagged] for details.

Adding an explicit `repr` to an enum suppresses the null-pointer
optimization.

These reprs have no effect on a struct.




# repr(packed)

`repr(packed)` forces Rust to strip any padding, and only align the type to a
byte. This may improve the memory footprint, but will likely have other negative
side-effects.

In particular, most architectures *strongly* prefer values to be aligned. This
may mean the unaligned loads are penalized (x86), or even fault (some ARM
chips). For simple cases like directly loading or storing a packed field, the
compiler might be able to paper over alignment issues with shifts and masks.
However if you take a reference to a packed field, it's unlikely that the
compiler will be able to emit code to avoid an unaligned load.

**[As of Rust 2018, this still can cause undefined behavior.][ub loads]**

`repr(packed)` is not to be used lightly. Unless you have extreme requirements,
this should not be used.

This repr is a modifier on `repr(C)` and `repr(rust)`.




# repr(align(n))

`repr(align(n))` (where `n` is a power of two) forces the type to have an
alignment of *at least* n.

This enables several tricks, like making sure neighboring elements of an array
never share the same cache line with each other (which may speed up certain
kinds of concurrent code).

This is a modifier on `repr(C)` and `repr(rust)`. It is incompatible with
`repr(packed)`.





[reference]: https://github.com/rust-rfcs/unsafe-code-guidelines/tree/master/reference/src/representation.md
[drop flags]: drop-flags.html
[ub loads]: https://github.com/rust-lang/rust/issues/27060
[`UnsafeCell`]: ../std/cell/struct.UnsafeCell.html
[rfc-transparent]: https://github.com/rust-lang/rfcs/blob/master/text/1758-repr-transparent.md
[really-tagged]: https://github.com/rust-lang/rfcs/blob/master/text/2195-really-tagged-unions.md
[rust-bindgen]: https://rust-lang.github.io/rust-bindgen/
[cbindgen]: https://github.com/eqrion/cbindgen
