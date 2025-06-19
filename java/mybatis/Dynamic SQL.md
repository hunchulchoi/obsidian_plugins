MyBatis Dynamic SQL은 런타임에 조건에 따라 SQL 문을 동적으로 생성하거나 수정하는 강력한 기능입니다. 이를 통해 복잡한 조건부 쿼리를 효율적으로 작성하고 관리할 수 있습니다[1][2].

## **동적 SQL의 핵심 태그**

### **if 태그**
가장 기본적인 조건문으로, 특정 조건이 참일 때만 SQL 구문을 포함시킵니다[3][4]:

```xml
<select id="getRecByName" parameterType="Student" resultType="Student">
    SELECT * FROM STUDENT
    <if test="name != null">
        WHERE name LIKE #{name}
    </if>
</select>
```

여러 조건을 조합할 수도 있습니다:
```xml
<select id="getRecByName_Id" parameterType="Student" resultType="Student">
    SELECT * FROM STUDENT
    <if test="name != null">
        WHERE name LIKE #{name}
    </if>
    <if test="id != null">
        AND id LIKE #{id}
    </if>
</select>
```

### **choose, when, otherwise 태그**
Java의 switch-case문과 유사한 다중 조건 분기를 제공합니다[2][5]:

```xml
<select id="selectList" parameterType="map" resultMap="projectResultMap">
    SELECT PNO, PNAME, STA_DATE, END_DATE, STATE 
    FROM PROJECTS 
    ORDER BY
    <choose>
        <when test="orderCond == 'TITLE_ASC'">PNAME ASC</when>
        <when test="orderCond == 'TITLE_DESC'">PNAME DESC</when>
        <when test="orderCond == 'STARTDATE_ASC'">STA_DATE ASC</when>
        <otherwise>PNO DESC</otherwise>
    </choose>
</select>
```

## **조건 특화 엘리먼트**

### **where 태그**
WHERE 절을 동적으로 생성하며, 내부 조건이 존재할 때만 WHERE 키워드를 추가하고 불필요한 AND/OR를 자동으로 제거합니다[2][6]:

```xml
<select id="selectUser" resultType="userDto">
    SELECT * FROM tb_user
    <where>
        <if test="userId != null">AND user_id = #{userId}</if>
        <if test="userName != null">AND user_name = #{userName}</if>
    </where>
</select>
```

### **set 태그**
UPDATE 문의 SET 절을 동적으로 생성하며, 불필요한 콤마를 자동으로 제거합니다[2][5]:

```xml
<update id="update" parameterType="map">
    UPDATE PROJECTS
    <set>
        <if test="title != null">PNAME = #{title},</if>
        <if test="content != null">CONTENT = #{content},</if>
        <if test="startDate != null">STA_DATE = #{startDate},</if>
        <if test="endDate != null">END_DATE = #{endDate},</if>
    </set>
    WHERE PNO = #{no}
</update>
```

### **trim 태그**
가장 유연한 태그로, 접두어/접미어 추가 및 불필요한 문자열 제거를 세밀하게 제어할 수 있습니다[2][7]:

```xml
<select id="selectUser" resultType="userDto">
    SELECT * FROM tb_user
    WHERE
    <trim prefixOverrides="AND ">
        <if test="userId != null">AND user_id = #{userId}</if>
        <if test="userName != null">AND user_name = #{userName}</if>
    </trim>
</select>
```

## **반복 처리**

### **foreach 태그**
컬렉션이나 배열을 순회하며 SQL을 생성합니다. IN 절 구성에 특히 유용합니다[2][6]:

```xml
<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT * FROM POST P
    WHERE ID IN
    <foreach item="item" index="index" collection="list"
             open="(" separator="," close=")">
        #{item}
    </foreach>
</select>
```

**foreach 속성 설명**:
- **item**: 각 항목을 참조할 변수명
- **index**: 인덱스 값을 참조할 변수명  
- **collection**: 순회할 컬렉션 (List, Array 등)
- **open**: 시작 문자열
- **close**: 종료 문자열
- **separator**: 구분자

## **동적 SQL 작성 규칙**

동적 SQL을 작성할 때 다음 규칙을 준수해야 합니다[1]:

- SQL 명령어와 MyBatis 태그 사이에는 공백을 포함해야 합니다
- 모든 MyBatis 태그는 반드시 닫는 태그로 끝나야 합니다
- 복잡성을 줄이기 위해 쿼리를 간결하게 유지하고 주석을 포함하는 것이 좋습니다

## **DBMS별 차이점**

LIKE 문 사용 시 DBMS별로 문법이 다르므로 주의가 필요합니다[8]:

**MySQL**:
```xml
WHERE COLUMN_NAME LIKE CONCAT('%', #{searchKeyword}, '%')
```

**Oracle**:
```xml
WHERE COLUMN_NAME LIKE '%'||#{searchKeyword}||'%'
```

**MS-SQL**:
```xml
WHERE COLUMN_NAME LIKE '%' + #{searchKeyword} + '%'
```

## **실제 활용 예제**

검색 기능을 구현하는 종합적인 예제입니다:

```xml
<select id="searchBooks" parameterType="map" resultType="Book">
    SELECT book_id, title, author, price, category
    FROM books
    <where>
        <if test="title != null and title != ''">
            AND title LIKE CONCAT('%', #{title}, '%')
        </if>
        <if test="author != null and author != ''">
            AND author LIKE CONCAT('%', #{author}, '%')
        </if>
        <if test="minPrice != null">
            AND price >= #{minPrice}
        </if>
        <if test="maxPrice != null">
            AND price <= #{maxPrice}
        </if>
        <if test="categories != null and categories.size() > 0">
            AND category IN
            <foreach item="category" collection="categories" 
                     open="(" separator="," close=")">
                #{category}
            </foreach>
        </if>
    </where>
    <choose>
        <when test="orderBy == 'price_asc'">ORDER BY price ASC</when>
        <when test="orderBy == 'price_desc'">ORDER BY price DESC</when>
        <when test="orderBy == 'title'">ORDER BY title ASC</when>
        <otherwise>ORDER BY book_id DESC</otherwise>
    </choose>
</select>
```

MyBatis Dynamic SQL을 통해 복잡한 조건부 쿼리를 간결하고 유지보수하기 쉬운 형태로 작성할 수 있으며, 런타임에 필요한 조건만을 포함하는 최적화된 SQL을 생성할 수 있습니다[6][4].

Sources
[1] [마이바티스(MyBatis)] 동적 SQL https://gdngy.tistory.com/159
[2] [MyBatis]MyBatis 동적 쿼리 문법 정리(if, choose, trim, where, set ... https://blog.naver.com/hj_kim97/222651531009
[3] MyBatis Dynamic SQL - Tutorialspoint https://www.tutorialspoint.com/mybatis/mybatis_dynamic_sql.htm
[4] [Mybatis] 동적 쿼리에 대해 알아보자 - 자바랑 썸타는중 https://gwamssoju.tistory.com/101
[5] MyBatis 동적 SQL - <choose>와 <set>을 사용하여 동적 SQL 만들기 https://atoz-develop.tistory.com/entry/MyBatis-%EB%8F%99%EC%A0%81-SQL-choose%EC%99%80-set%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-%EB%8F%99%EC%A0%81-SQL-%EB%A7%8C%EB%93%A4%EA%B8%B0
[6] 동적 SQL - 마이바티스 3 - MyBatis.org https://mybatis.org/mybatis-3/ko/dynamic-sql.html
[7] [Java] MyBatis 이해하기 -2 : 동적 SQL 태그(if, choose, trim, where ... https://adjh54.tistory.com/441
[8] [MyBatis] 동적 쿼리 LIKE 문 삽질 주의 - 오늘은 룡 - 티스토리 https://dongram.tistory.com/12
[9] MyBatis Dynamic SQL https://mybatis.org/mybatis-dynamic-sql/docs/introduction.html
[10] [Spring] mybatis의 동적 쿼리 - velog https://velog.io/@ryuneng2/Spring-mybatis%EC%9D%98-%EB%8F%99%EC%A0%81%EC%BF%BC%EB%A6%AC
[11] [MyBatis] 동적 SQL - 태옹 - 티스토리 https://taetoungs-branch.tistory.com/149
[12] [spring] mybatis/동적쿼리 - velog https://velog.io/@cutehypretty/spring-mybatis%EB%8F%99%EC%A0%81%EC%BF%BC%EB%A6%AC
[13] mybatis select 태그 정리 - 인포꿀팁 - 티스토리 https://back-end-developer.tistory.com/entry/mybatis-select-%ED%83%9C%EA%B7%B8-%EC%A0%95%EB%A6%AC
[14] [MyBatis] RDBMS별 동적 쿼리 LIKE 문 작성시 주의사항 : 네이버 블로그 https://blog.naver.com/nbb_pinetree?Redirect=Log&logNo=221549663506
[15] [MyBatis] 동적 쿼리 <trim> 개념 및 문법 총 정리 -.java의 개발일기 https://java119.tistory.com/103
[16] [JAVA] MyBatis - 동적 SQL - Amy IT - 티스토리 https://amy-it.tistory.com/66
[17] [MyBatis] 동적 SQL : 네이버 블로그 https://blog.naver.com/hoisho/223073452818
[18] MyBatis Dynamic SQL - Unvus Docs https://dev.unvus.com/unvuscore/core/dynamic-sql.html
