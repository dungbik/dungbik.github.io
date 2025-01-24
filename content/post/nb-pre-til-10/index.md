---
title: 사전캠프 10일차
description:
slug: nb-pre-til-10
date: 2025-01-24 00:00:00+0000
image:
weight: 1
categories:
    - til
tags:
    - 내일배움캠프
    - 사전캠프
---

## 사전캠프 달리기반 Java 응용하기

## Lv. 1 랜덤 닉네임 생성기
``` java
package level1;

import java.util.List;
import java.util.Random;

public class NicknameCreator {
    private final Random rand = new Random();
    private final String[] firstValues = {"기철초풍", "멋있는", "재미있는"};
    private final String[] secondValues = {"도전적인", "노란색의", "바보같은"};
    private final String[] thirdValues = {"돌고래", "개발자", "오랑우탄"};

    public String createNickname() {
        List<String> wordList = List.of(firstValues[rand.nextInt(firstValues.length)],
                secondValues[rand.nextInt(secondValues.length)],
                thirdValues[rand.nextInt(thirdValues.length)]);
        return String.join(" ", wordList);
    }

    public static void main(String[] args) {
        NicknameCreator nicknameCreator = new NicknameCreator();
        String nickname = nicknameCreator.createNickname();
        System.out.println(nickname);
    }
}
``` 

### Lv2. 스파르타 자판기
``` java
package level2;

public record Drink(
        String name,
        int price
) {
    @Override
    public String toString() {
        return String.format("%s %s원", name, NumberUtil.formatNumber(price));
    }
}
```
``` java
package level2;

import java.text.DecimalFormat;

public class NumberUtil {
    private static final DecimalFormat decFormat = new DecimalFormat("###,###");

    public static String formatNumber(int value) {
        return decFormat.format(value);
    }
}

```
``` java
package level2;

import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class VendingMachine {
    private List<Drink> drinks;

    public static void main(String[] args) {
        VendingMachine vm = new VendingMachine();
        vm.initDrinkInfo();
        vm.showDrinkInfo();
        vm.startMachine();
    }

    public void initDrinkInfo() {
        drinks = new ArrayList<>();
        drinks.add(new Drink("사이다", 1700));
        drinks.add(new Drink("콜라", 1900));
        drinks.add(new Drink("식혜", 2500));
        drinks.add(new Drink("솔의눈", 3000));
    }

    public void showDrinkInfo() {
        drinks.forEach(System.out::println);
    }

    public Drink getDrink(String drinkName) {
        return drinks.stream()
                .filter(d -> d.name().equals(drinkName))
                .findFirst()
                .orElse(null);
    }

    public void startMachine() {
        Scanner s = new Scanner(System.in);
        while (true) {
            System.out.print("구매할 음료 : ");
            String name = s.nextLine();
            if (name.isEmpty()) {
                if (s.hasNext()) {
                    s.nextLine();
                }
                continue;
            }

            Drink drink = getDrink(name);
            if (drink == null) {
                break;
            }
            
            System.out.print("지불할 금액 : ");
            int price = s.nextInt();
            if (price < drink.price()) {
                System.out.println("돈이 부족합니다.");
                continue;
            }

            String remainedMoney = NumberUtil.formatNumber(drink.price() - price);
            System.out.printf("잔액 %s%n", remainedMoney);
        }
    }
}

```

### Lv3. 단어 맞추기 게임
``` java
package level3;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.Scanner;

public class WordGuessingGame {

    private static String[] words = {"airplane", "apple", "arm", "bakery", "banana", "bank", "bean", "belt", "bicycle", "biography", "blackboard", "boat", "bowl", "broccoli", "bus", "car", "carrot", "chair", "cherry", "cinema", "class", "classroom", "cloud", "coat", "cucumber", "desk", "dictionary", "dress", "ear", "eye", "fog", "foot", "fork", "fruits", "hail", "hand", "head", "helicopter", "hospital", "ice", "jacket", "kettle", "knife", "leg", "lettuce", "library", "magazine", "mango", "melon", "motorcycle", "mouth", "newspaper", "nose", "notebook", "novel", "onion", "orange", "peach", "pharmacy", "pineapple", "plate", "pot", "potato", "rain", "shirt", "shoe", "shop", "sink", "skateboard", "ski", "skirt", "sky", "snow", "sock", "spinach", "spoon", "stationary", "stomach", "strawberry", "student", "sun", "supermarket", "sweater", "teacher", "thunderstorm", "tomato", "trousers", "truck", "vegetables", "vehicles", "watermelon", "wind"};

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        Random rand = new Random();
        String selectedWord = words[rand.nextInt(words.length)];
        StringBuilder inputWord = new StringBuilder("_".repeat(selectedWord.length()));
        List<String> usedChars = new ArrayList<>();
        System.out.println(inputWord);

        for (int retry = 0; retry < 9; retry++) {
            String input = sc.nextLine();
            while (input.length() != 1 || !input.matches("^[a-zA-Z]*$") || usedChars.contains(input)) {
                input = sc.nextLine();
            }

            usedChars.add(input);
            if (selectedWord.contains(input)) {
                for (int i = 0; i < selectedWord.length(); i++) {
                    if (selectedWord.charAt(i) == input.charAt(0)) {
                        inputWord.setCharAt(i, input.charAt(0));
                    }
                }
            }
            System.out.println(inputWord);
            if (inputWord.toString().equals(selectedWord)) {
                System.out.println("플레이어의 승리");
                break;
            }
        }

    }
}
```

---

## 코드카타
* 1번 - [모음사전](https://school.programmers.co.kr/learn/courses/30/lessons/84512), Lv. 2
* 2번 - [호텔 방 배정](https://school.programmers.co.kr/learn/courses/30/lessons/64063), Lv. 4
