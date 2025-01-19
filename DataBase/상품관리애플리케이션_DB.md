# **상품 관리 애플리케이션**
> [리스트로 관리되는 상품관리애플리케이션](https://github.com/dbalsk/TIL/blob/main/SpringBoot/%EC%83%81%ED%92%88%EA%B4%80%EB%A6%AC%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98.md)을 수정하여 **데이터베이스로 관리되는 상품관리애플리케이션**을 만들어본다. 이를 통해 **데이터베이스의 사용방법**과 **데이터베이스 연동**에 대해 학습해보자.
  
## 데이터베이스
도커 데스크톱에서 MySQL 도커 컨테이너로 직접 접속하여 **[데이터베이스](https://github.com/dbalsk/TIL/blob/main/DataBase/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4.md)** 사용 

**✔ 스키마 추가**   
```
    CREATE SCHEMA product_management;
```

**✔ 테이블 추가**  
product_management 스키마에 테이블 생성
```
    CREATE TABLE products (
        id BIGINT PRIMARY KEY NOT NULL AUTO_INCREMENT,
        name VARCHAR(100) NOT NULL,
        price INT NOT NULL,
        amount INT NOT NULL
    );
```
- 자바 코드에서의 Product 클래스의 필드와 **동알한 이름**의 컬럼(Column)
- id 컬럼
    - 타입: **BIGINT** (자바의 LONG과 동일)
    - **PRIMARY KEY**: 값 중복 x / 인덱스 생성 (id 컬럼 기준으로 빠르게 조회 가능) 
    - **NOT NULL**: NULL 값 x
    - **AUTO_INCREMENT**: 상품 추가마다 해당 컬럼 1씩 증가
- name 컬럼
    - 타입: **VARCHAR** (문자열 저장하는 타입, 괄호 안은 문자열 길이)
    - **NOT NULL**: NULL 값 x
- price 컬럼
    - 타입: **INT**
    - **NOT NULL**: NULL 값 x
- amount 컬럼 
    - 타입: **INT**
    - **NOT NULL**: NULL 값 x

## 데이터베이스 연동
**✔ 의존성 추가**     
- **spring-boot-starter-jdbc**: 스프링 부트에서 제공해주는 JDBC 관련 의존성 모음
- **mysql-connector-java**: MySQL에 접속하기 위해 필요한 의존성

>+**JDBC**(Java Database Connectivity)  
자바에서 데이터베이스에 접속하는 기능을 제공하는 API 

**✔ 데이터베이스 접속정보 추가**    
```
//application.propoerties 파일에 입력

spring.datasource.url=jdbc:mysql://localhost:3306/product_management 
//데이터베이스의 URL 입력 
spring.datasource.username=root
//데이터베이스의 계정 아이디
spring.datasource.password=비밀번호
//데이터베이스의 비밀번호 (추후 암호화 필요 - jasypt 또는 Vault)
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
//데이터베이스 연결 시 사용할 드라이버
```

**✔ [커넥션풀]() 초기화**  
1. spring-boot-starter-jdbc 의존성 사용할 때 **커넥션풀 자동 생성** (첫번째 요청이 와야 생성)
2. **ApplicationRunner 의존성**을 빈으로 등록하여 생성해줌으로써 커넥션풀 생성 (첫번째 요청을 수행했기에)
    ```
    @Bean
	public ApplicationRunner runner(DataSource dataSource) {
		return args -> {
			//실행할 코드
			Connection connection = dataSource.getConnection();
		};
	}
    ```
3. 'HikariPool-1-Start completed' 메시지 발생 (성공적으로 커넥션풀이 생성되었다는 로그 메시지)

>+**ApplicationRunner**  
스프링부트 애플리케이션이 시작한 직후 실행하려는 코드를 추가하는 의존성.  
매개변수인 **DataSourse**(데이터베이스와의 연결을 담당하는 인터페이스)로 데이터베이스와의 커넥션을 가져올 수 있음. (커넥션을 가져오는 것이 성공한다면 데이터베이스와 연결 성공한 것)

**✔ 레포지토리 수정**  
기존의 ListProductRepository를 **DatabaseProductRepository**로 새롭게 구현
- **JdbcTemplate 의존성 주입**
- 기존에 구현했던 메소드 가져오기 

</br>

## 상품 추가 기능 구현
1. (INSERT SQL 쿼리를 보내기 위해) Product 클래스에 각 필드의 **getter** 추가 (위 쿼리에서는 id의 getter 필요x)
2. (데이터베이스에서 id 값을 받아오고 지정하기 위해) JdbcTemplate 의존성을 **NamedParameterJdbcTemplate 의존성**으로 변경 (JdbcTemplate로 상품 id를 받아오는 코드는 좀 더 복잡)
3. **SqlParameterSource**로 객체를 매핑
 > **BeanPropertySqlParameterSource** 사용 (product의 getter를 통해 sql 쿼리의 매개변수를 매핑해주는 객체) 	 
4. **KeyHolder** 사용 (INSERT 작업 후 데이터베이스에 생성된 키(id)를 가져오는 객체)
5. **update** 메소드 호출
 
```ruby
    public Product add(Product product){
        KeyHolder keyHolder = new GeneratedKeyHolder();
        //keyHolder를 인자로 넘겨주면 id가 담겨옴. 
        SqlParameterSource namedParameter = new BeanPropertySqlParameterSource(product);
        //namedParameter를 인자로 넘겨주는 것만으로 (getter 메소드를 직접 사용하지 않고) 매개변수 매핑 가능 

        namedParameterJdbcTemplate
                .update("INSERT INTO products (name, price, amount) VALUES (:name, :price, :amount)",
                        namedParameter, keyHolder);

        Long generatedId = keyHolder.getKey().longValue(); //Long타입의 id 가져오기
        product.setId(generatedId); //해당 id를 Product에 지정 

        return product;
    }
```

>+**JdbcTemplate**  
JdbcTemplate.update 메소드의 인자로 들어간 SQL 쿼리는 물음표로 매개변수를 매핑함. (**NamedParameterJdbcTemplate**은 매개변수의 이름을 통해 값을 매핑) 매개변수의 수가 많을 경우 물음표 순서가 헷갈릴 수 있다는 단점이 있음.   
>```
>JdbcTemplate.update("INSERT INTO products (name, price, amount) VALUES (?, ?, ?)", product.getName(), product.getPrice(), product.getAmount());
>```

## 상품 조회 기능 구현 
**✔ 상품 번호로 조회 기능 (findById 메소드)**
1. **SqlParameterSource**로 id를 매핑
> **MapSqlParameterSource** 사용 (Map 형태로 Key-Value 형태를 매핑해주는 객체)  
> (BeanPropertySqlParameterSource는 객체를 매핑하기에 id 매핑으로는 위 객체가 적절)
2. queryForObject 메소드 호출 (하나의 Product 조회)
> 위 메소드의 인자로 **BeanPropertyRowMapper** 받음 (조회된 데이터를 자바의 인스턴스로 변환해주는 객체)  
3. **BeanPropertyRowMapper**의 작동을 위해 Product의 각 필드에 대한 setter 추가 

```ruby
    public Product findByID(Long id){
        SqlParameterSource namedParameter = new MapSqlParameterSource("id", id);

        Product product = namedParameterJdbcTemplate.queryForObject(
                "SELECT id, name, price, amount FROM products WHERE id=:id", 
                namedParameter, 
                new BeanPropertyRowMapper<>(Product.class)
                //BeanPropertyRowMapper가 변환을 수행하기위해 2가지 과정 필요
                //1. 인자가 없는 Product 생성자로 Product 인스턴스 생성 2. setter로 각 필드 초기화
        );
        return product;
    }
```

**✔ 상품 전체 조회 기능 (findAll 메소드)**  
1. query 메소드 호출 (전체 Product 리스트 조회)

```ruby
    public List<Product> findAll(){
        // 전체목록 조회에는 매개변수가 필요없기에 MapSqlParameterSource 필요x
        List<Product> products = namedParameterJdbcTemplate.query(
                "SELECT * FROM products",
                new BeanPropertyRowMapper<>(Product.class)
        );
        return products;
    }
```

**✔ 문자열로 검색 기능 (findByNameContaining 메소드)**  
1. **SqlParameterSource**로 name을 매핑
>**MapSqlParameterSource** 사용 
2. query 메소드 호출 (name이 포함되는 Product 리스트 조회)

```ruby
    public List<Product> findByNameContaining(String name){
        SqlParameterSource namedParameter = new MapSqlParameterSource("name", "%"+name+"%");
        //"&"를 앞뒤로 붙여 name이 앞,중간,뒤에 포함되는 경우도 검색 (안붙이면 정확히 일치하는 값 검색)
        
        List<Product> products = namedParameterJdbcTemplate.query(
                "SELECT * FROM products WHERE name LIKE :name",
                namedParameter,
                new BeanPropertyRowMapper<>(Product.class)
        );
        return products;
    }
```

## 상품 수정 기능 구현
1. (UPDATE SQL 쿼리를 보내기 위해) Product 클래스에 **id의 getter** 추가 (나머지 필드의 getter는 이미 추가)
2. **SqlParameterSource**로 객체를 매핑
> **BeanPropertySqlParameterSource** 사용
3. update 메소드 호출

```ruby 
    public Product update(Product product) {
        SqlParameterSource namedParameter = new BeanPropertySqlParameterSource(product);

        namedParameterJdbcTemplate.update("UPDATE products SET name=:name, price=:price, amount=:amount WHERE id=:id",
                namedParameter);

        return product;
    }
```

## 상품 삭제 기능 구현
1. **SqlParameterSource**로 id를 매핑
> **MapSqlParameterSource** 사용
2. update 메소드 호출

```ruby
    public void delete(Long id) {
        SqlParameterSource namedParameter = new MapSqlParameterSource("id", id);

        namedParameterJdbcTemplate.update(
                "DELETE FROM products WHERE id=:id",
                namedParameter
        );
    }
```

>+namedParameterJdbcTemplate의 대표 메소드  
>1. **query()**: 조회 SQL을 실행하여 객체 리스트를 반환 
>2. **queryForObject()**: 조회 SQL을 실행하여 하나의 객체를 반환  
>3. **update()**: 삽입, 수정, 삭제 SQL을 실행  

</br>

## 참고자료
[이것이백엔드개발이다](https://product.kyobobook.co.kr/detail/S000211834105)  
