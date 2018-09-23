# 레퍼런스

러스트엔 두 종류의 레퍼런스가 있습니다.

* 공유된 레퍼런스: `&`
* 수정 가능한(뮤터블<sub>mutable</sub>) 레퍼런스: `&mut`

각각은 아래 규칙을 지킵니다.

* 레퍼런스는 지시 대상보다 오래 살 수 없습니다.
* 뮤터블 레퍼런스에 다른 이름을 붙일 수 없습니다.

이게 끝입니다. 이게 레퍼런스가 지키는 전부입니다.

물론, *다른 이름을 붙인다*는 게 무슨 의민지 정의가 필요합니다.

```text
error[E0425]: cannot find value `aliased` in this scope
 --> <rust.rs>:2:20
  |
2 |     println!("{}", aliased);
  |                    ^^^^^^^ not found in this scope

error: aborting due to previous error
```

안타깝게도, 러스트는 이름 붙이기 모델을 딱히 정의하지 않았습니다. 🙀

러스트 개발진이 언어 의미론을 정립하기 전에 다음 장에서 일반적으로 이름
붙이기가 무엇인지, 왜 중요한지 알아봅시다.
