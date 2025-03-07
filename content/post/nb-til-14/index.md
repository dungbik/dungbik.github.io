---
title: 내일배움캠프 - 14일차
description:
slug: nb-til-14
date: 2025-03-07 00:00:00+0000
image:
weight: 1
categories:
  - til
tags:
  - 내일배움캠프
  - 본캠프
---

## ch.2 키오스크 과제 트러블 슈팅

### 개요
키오스크는 전에 수행했던 과제와 다르게 단계가 나누어져 있다.  
카테고리 선택 단계, 메뉴 선택 단계 등 여러 개의 단계로 이루어져 있는데, 현재 단계가 무슨 단계인지 알 수 있는 것이 필요했다.

### 트러블 슈팅

#### 배경
```java

private int curStage = 0;

public void start() {
    
    while (curStage < 7) {
        switch (curStage) {
            case 0 -> { ... }
            case 1 -> { ... }
            case 2 -> { ... }
            case 3 -> { ... }
        }
    }
    
}

public void order() {
    
    ...
    
    // 주문이 끝나서 카테고리 선택 기능으로 돌아가야함
    curStage = 0;
}
```

#### 발단

배경의 코드와 같이 현재 상태를 숫자로서 나타내게 코드를 작성을 하였다.  
현재 단계를 이동시킬 때, 숫자로 구분을 하니 다른 기능 단계를 실수로 적어 기능이 이상하게 구현되는 문제가 있었다.  

예를 들면, 주문이 끝난 후 카테고리 선택 기능으로 돌아가야 하는데 메뉴 선택을 하는 기능으로 동작되는 이런 오류들이 발생하였다.

#### 전개

```java

private static final int SELECT_CATEGORY = 0;
...
private static final int EXIT = 7;

```
이를 해결하기 위해 처음에 생각한 방법은 상수로써 관리를 하는 것이였다.  
이렇게 상수로 관리를 하니 전에 같이 실수를 하는 일은 없었다. 
하지만 실시간 강의 때, 해당 상수들이 집합으로써 묶이는 경우에는 Enum 을 사용하는 것도 좋은 방법이라고 들어서 이를 이용하여 코드를 구현하였다.

#### 결말

```java
/**
 * 키오스크 시스템의 메인 실행 루프를 시작합니다.
 * 이 메서드는 현재 단계와 사용자 상호작용에 따라 키오스크 운영의 다양한 단계를 지속적으로 전환합니다.
 * <p>
 * 이 메서드는 상태 변화를 관리하고 키오스크 워크플로우와 관련된 특정 작업을 호출합니다:
 * - SELECT_CATEGORY: 사용자가 카테고리를 선택할 수 있도록 지원합니다.
 * - SELECT_MENU_ITEM: 선택된 카테고리 내에서 메뉴 항목 선택을 처리합니다.
 * - CONFIRM_ADD_TO_CART: 선택된 항목을 장바구니에 추가할지 사용자 확인을 요청합니다.
 * - ADD_TO_CART: 선택된 항목을 장바구니에 추가합니다.
 * - CONFIRM_ORDER: 주문을 진행할지 사용자 확인을 요청합니다.
 * - CANCEL_ORDER: 진행 중인 주문을 취소하고 필요한 상태를 재설정합니다.
 * - REMOVE_CART_ITEM: 사용자가 장바구니에서 항목을 제거할 수 있도록 합니다.
 * - SELECT_BENEFIT: 사용자가 주문을 완료하기 전에 적용 가능한 혜택을 선택할 수 있도록 합니다.
 * - ORDER: 주문을 확정하고, 할인을 적용하며, 거래를 완료합니다.
 * <p>
 * 현재 단계가 Stage.EXIT로 전환되면 메서드는 종료됩니다.
 */
public void start() {
    while (curStage != Stage.EXIT) {
        switch (curStage) {
            case SELECT_CATEGORY -> selectCategory();
            case SELECT_MENU_ITEM -> selectMenuItem();
            case CONFIRM_ADD_TO_CART -> confirmAddToCart();
            case ADD_TO_CART -> addToCart();
            case CONFIRM_ORDER -> confirmOrder();
            case CANCEL_ORDER -> cancelOrder();
            case REMOVE_CART_ITEM -> removeCartItem();
            case SELECT_BENEFIT -> selectBenefit();
            case ORDER -> order();
        }
    }
}
```

최종적으로 위와 같은 코드를 작성하였다.

### 마무리

이번 트러블 슈팅을 요약하면 다음과 같습니다.

1. 코드를 작성하다보면 상태 혹은 단계, 종류 등을 저장하고 사용해야할 때가 있습니다.
2. 이 때 단순히 문자열이나 숫자로 사용하는 것은 매우 안좋은 방법입니다.
   - 가독성이 떨어지고 실수가 많이 생김
3. 이럴 때는 상수 혹은 Enum 을 사용하는 것이 방법이 될 수 있습니다.
   - 집합이 가능하다면 Enum, 외의 경우에는 상수 
