---
title: 엔티티에 없는 정보까지 필요한 경우의 고민
description:
slug: til-2
date: 2025-03-20 00:00:00+0000
image:
weight: 1
categories:
  - til
tags:
  - 스프링
  - 트러블 슈팅
---

## 개요
일정 관리 앱을 개발하던 중에 발생한 문제이다.

일정을 조회하는데 사용자 정보까지 필요한 상황이다.

## 트러블 슈팅

### 배경
현재 일정 엔티티에는 사용자의 정보는 없고 사용자의 id 필드만 가지고 있다.

### 발단

일정을 조회했을 때 id 뿐만이 아닌 사용자의 다른 정보들도 응답에 넣어주려고 한다.

현재는 일정을 조회했을 때 사용자에 관한 정보는 사용자의 id만 알 수 있는 상태이다.

### 전개

이를 해결하기 위해서는 두가지의 방법이 있을 것 같다.

1. JOIN 을 통해 한번의 조회문으로 사용자의 id가 일치하는 사용자의 정보를 같이 조회한다.
2. 일정을 조회하고 사용자의 id로 사용자 정보를 한번 더 조회해서 병합한다.

### 결말

```java
/**
 * 검색 쿼리를 동적으로 생성
 * @param cond 검색 조건
 * @return 검색을 수행하는 쿼리문
 */
private String createSearchQuery(ScheduleSearchCond cond) {
    String sql = "SELECT DISTINCT s.id, u.email, u.name, s.task, s.created_at, s.updated_at FROM schedule s JOIN user u ON u.id = s.user_id";

    if (StringUtils.hasText(cond.getUsername())) {
        sql += " AND u.name like concat('%', :username, '%')";
    }

    if (cond.getUpdatedAt() != null) {
        sql += " WHERE DATE(s.updated_at) = :updatedAt";
    }

    if (cond.getUserId() != null) {
        sql += " AND s.user_id = :userId";
    }

    sql += " ORDER BY s.updated_at DESC LIMIT :limit OFFSET :offset";
    return sql;
}

/**
 * 작성자 정보를 포함한 일정 정보를 만드는 Mapper
 * @return 작성자 정보를 포함한 일정 정보를 만드는 Mapper
 */
private RowMapper<ScheduleResponse> scheduleResponseRowMapper() {
    return ((rs, rowNum) -> {
        ScheduleResponse response = new ScheduleResponse();
        response.setScheduleId(rs.getLong("s.id"));
        response.setUser(new UserResponse(rs.getLong("u.id"), rs.getString("u.email"), rs.getString("u.name")));
        response.setTask(rs.getString("s.task"));
        response.setCreatedAt(rs.getTimestamp("s.created_at").toLocalDateTime());
        response.setUpdatedAt(rs.getTimestamp("s.updated_at").toLocalDateTime());
        return response;
    });
}
```

최종적으로 위와 같은 코드를 작성하였다.

JOIN 을 통해 사용자의 정보까지 같이 조회한 후 사용자의 정보를 포함한 일정 정보를 얻을 수 있도록 하였다.

## 마무리

이번 트러블 슈팅을 요약하면 다음과 같습니다.

1. 웹 애플리케이션을 작업하다 보면 다수의 엔티티를 병합하여 사용해야 하는 경우가 생깁니다.
2. 이 때 우리가 생각할 수 있는 방법에는 JOIN 활용 혹은 따로 조회 후 병합 등 여러가지 방법이 있을 수 있습니다. 
3. 이번 경우에는 2번 호출하는 것 보단 1번의 호출로 조회할 수 있는 것이 좋을 것 같아 JOIN 을 활용하였습니다.
