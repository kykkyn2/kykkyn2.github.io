---
layout: post
title: "Postgresql UpdateAt 설정 하는 방법"
author: "kykkyn2"
---


### Postgresql UpdateAt 설정 하기

DB 사용시 대부분의 테이블 마다 CreateAt 및 UpdateAt 컬럼이 존재 한다.

테이블 데이타 Row 생성된 시간 및 일부 컬럼이 수정된 시간을 나타내기 위한 컬럼이다.

Mysql, Maria 같은 경우는 `CURRENT_TIMESTAMP` 로 간단하게 셋팅이 가능하다. 하지만 Postgresql 은 위와 같은 방법으로는 동작 하지 않는다.

### 트리거 구성 하즈아

```sql
    CREATE FUNCTION upsert_date() RETURNS TRIGGER
      LANGUAGE plpgsql
    AS $$
    BEGIN
      NEW.updated_at = now();
      RETURN NEW;
    END;
    $$;

    CREATE TRIGGER trigger_upsert_date BEFORE UPDATE ON 테이블 이름 FOR EACH ROW EXECUTE PROCEDURE upsert_date();
```
테이블 구성

![shot1](https://user-images.githubusercontent.com/5660626/50833692-737ae700-1395-11e9-8f81-af65337f441b.png)

적용 예시

맨 아래 컬럼 일부를 수정 했더니 update_at 컬럼이 자동으로 업데이트 된것을 볼 수 있다.

![shot2](https://user-images.githubusercontent.com/5660626/50833760-a6bd7600-1395-11e9-8e64-00078505bbb5.png)

위와 같이 FUNC 를 만들어서 사용하면 간단하게 사용 할 수 있습니다. 테이블 만들때 위에 TRIGGER 만 적용 하면 됩니다. 참 쉽죠 :)

### 마무리

반복 작업은 항상 FUNC 을 만들어 사용하면 손 쉽게 작업 할 수 있습니다.

저도 위와 같이 설정을 해두고 엄청 유용하게 사용하고 있습니다.

다들 즐거운 코딩 하세요 :)
