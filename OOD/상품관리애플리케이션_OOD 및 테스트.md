# **상품 관리 애플리케이션**
**[데이터베이스로 관리되는 상품관리애플리케이션](https://github.com/dbalsk/TIL/blob/main/DataBase/%EC%83%81%ED%92%88%EA%B4%80%EB%A6%AC%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98_DB.md)** 에 **객체지향적 설계 추가** 및 **리팩토링**을 하여 프로젝트 완성
  
## 추상화
>클래스를 추상화함으로써 유연성을 높이는 설계로 개선해보자. 

**✔ 애플리케이션 실행환경**   
기존의 상품관리애플리케이션은 실행환경에 따라 SimpleProductService(서비스 클래스)에 서로 다른 빈이 주입되어야 하는 상황

- 테스트 환경 (로컬 개발환경) -> **ListProductRepository** 빈 주입 (리스트 기반 리포지토리 클래스)
- 서비스 환경 -> **DatabaseProductRepository** 빈 주입 (데이터베이스 기반 리포지토리 클래스)

**✔ 인터페이스에 의존 (추상화)**   
인터페이스에 의존하도록 구현함으로써 실행환경에 따라 동작을 정의하도록 구현 

- 인터페이스 생성: 두 리포지토리 클래스의 인터페이스로 **ProductRepository** 생성 

- 인터페이스 위치: **도메인 계층**  
-> 두 리포지토리 클래스가 위치하는 **인프라스트럭처 계층**에 위치할 경우 **도메인 주도 설계의 의존성 방향에 위배**됨. (다른 모든 계층은 인프라스트럭처 계층에 의존하면 안됨)

- 의존성 방향: **인터페이스(추상적 존재)로 의존성 방향**이 모임. 
    - **서비스 클래스**: 인터페이스를 사용하기에 인터페이스에 의존
    - **리포지토리 클래스**: 인터페이스를 구현하기에 인터페이스에 의존  

- 인터페이스의 효과: 구체적인 존재가 아닌 **추상적인 존재에 의존**함으로써 애플리케이션의 **동작을 코드 변경 없이 실행 시점**에 결정 

**✔ 문제 해결**
- 문제1: 인터페이스에 두 클래스의 빈 중, 어떤 것을 넣을지 판단 불가 
    - 해결방법: **Spring Profiles** 사용 (테스트/서비스 환경에 따라 다르게 적용할 수 있도록)
        - 각 구현체에 **@Profile 추가**    
        ```@Profile("test"), @Profile("prod")```   
        - **application.properties** 파일에서 어떤 Profile로 실행할지 지정    
        ```spring.profiles.active=test```  
        - 명령어로 Profile 명시할 경우 그에 따라 실행 (명령어로 지정하지 않는 경우 위 파일에 따라 실행)  

- 문제2: 테스트 환경에서 실행 불가   
  (jdbc 의존성에서 **데이터베이스 연결 관련 자동설정**을 진행하기에 데이터베이스를 사용하지 않는 경우 실행 불가)
    - 해결방법: **properties 파일 나누기**
      >[-**Profile**]에 따라 스프링 부트에서 사용될 설정파일을 자동으로 매핑해줌.
        - **application.properties** -> 기본 Profile로 test를 지정    
            (기본값은 test로 설정하는 것이 좋음. 프로젝트 다운로드할 경우 데이터베이스없이 바로 실행 가능하도록)
        - **application-test.properties** -> 데이터베이스 연결 자동설정 제외   
        - **application-prod.properties** -> 데이터베이스 연결정보
        - ApplicationRunner에 @Profile("prod") 추가

## 의존성 주입 / 의존성 역전 원칙 
> 추상화를 통해 구체적인 구현이 아닌 인터페이스에 의존하도록 설계했다.
이를 통해 적용할 수 있었던 **의존성 주입**과 **의존성 역전 원칙**에 대해 살펴보자.

**✔ 의존성 주입 (DI-Dependency Injection)**   
객체 간의 의존 관계를 외부에서 주입하는 설계 패턴 (의존성 주입 패턴)

- 의존성 주입 패턴을 따르지 않는 경우 (**의존성을 직접 생성**하는 경우) -> 추상화의 이점 사라짐      
  - 객체가 **강하게 결합**
  - 의존성 **유연하게 변경 불가** 
  - (위 프로젝트의 경우) 응용 계층에서 인프라스트럭처 계층으로 다시 **의존 생김**.   


- 의존성 주입의 장점   
  - 객체 간 **결합도 감소** (추상화에 의존하여 객체 간 의존도 적음)
  - **유연성** (실행시점에 의존성 변경 가능)
  - **확장성** (새로운 구현체가 추가되어도 수정 필요없음) 
  - 테스트 용이성 (목 객체로 외부 의존성 없이 단위 테스트 수행 가능)

**✔ 의존성 역전 원칙 (DIP-Dependency Inversion Principle)**  
고수준 컴포넌트가 저수준 컴포넌트에 의존하면 안됨 -> 둘다 **추상화에 의존**해야함

>+고수준/저수준  
**고수준** 예시 -> 비즈니스 로직 또는 전체적인 흐름을 담당.   ex) 응용 계층   
**저수준** 예시 -> 구체적인 구현 및 수행. ex) 데이터베이스 담당하는 리포지토리   
(하지만 고수준/저수준은 상대적인 기준이기에 두 대상을 비교할 경우 **도메인과 가까울수록 고수준**, **외부와 가까울수록 저수준**으로 나눠야함)

- 의존성 방향
  - 기존의 방향: 서비스 계층(고수준)이 인프라스트럭처 계층(저수준)에 의존 
  - 변경된 방향: **인터페이스(추상적 존재)로 의존성 방향**이 모임. (저수준이 고수준에 의존하는 방향으로 **역전**됨) 


-> 즉, **의존성 역전 원칙**을 구현하기 위해 **추상화**를 사용하여 구체적인 구현이 아닌 인터페이스에 의존하도록 설계하고, **의존성 주입**을 통해 해당 인터페이스의 구현체를 외부에서 주입받았다. 


## 리팩토링
>리팩토링을 통해 결과의 변경없이 코드의 구조를 개선해보고, 통합테스트/단위테스트를 사용하여 리팩토링이 정상적으로 완료됐는지 확인해보자.

**✔ 리팩토링 (Refactoring)**  
동일한 입력에 대해 **결과의 변경없이** 코드의 구조 개선


**✔ 테스트 코드**  
작성한 로직을 테스트하기 위한 코드 -> 리팩토링 후 기존과 동일하게 작동하는지 확인 가능  

- **통합테스트**: 2개 이상의 모듈(클래스or컴포넌트)을 검증하는 테스트
  - 여러 계층의 테스트 및 계층 간 연동성 확인 
  - 실행속도 느림
  - 외부 의존성 사용 (DB, 서버 등) -> 실제환경과 비슷한 테스트 
- **단위테스트**: 개별 모듈을 독립적으로 검증하는 테스트 
  - 특정 메소드/클래스의 테스트
  - 실행속도 빠름
  - 외부 의존성 x -> Mock 사용 

>**+모킹(mocking)**  
>Mock 객체를 만들어 의존성을 제거하고 독립적으로 테스트 

**✔ 리팩토링 및 테스트 코드 작성**    
- 리팩토링: ModelMapper 제거하고 Product-Dto 변환을 수행하는 메소드 구현 
  - Dto가 엔티티를 의존하도록 구현 (도메인 계층은 다른 계정 의존 x)
  - 생성자를 사용하여 구현 
  
```ruby
    //ProductDto에 두 메소드 모두 구현 

    //Dto -> 엔티티 변환
    public static Product toEntity(ProductDto productDto){
        Product product = new Product(
                productDto.getId(),
                productDto.getName(),
                productDto.getPrice(),
                productDto.getAmount()
        );

        return product;
    }

    //엔티티 -> Dto 변환
    public static ProductDto toDto(Product product){
        ProductDto productDto = new ProductDto(
                product.getId(),
                product.getName(),
                product.getPrice(),
                product.getAmount()
        );

        return productDto;
    }

```

- 통합테스트 
  - 테스트 코드1: add, findById 작동 테스트 
  - 테스트 코드2: findById 예외 발생 테스트

```ruby
@SpringBootTest //스프링부트를 실행하는 통합 테스트 (필요한 의존성을 빈으로 등록 및 주입)
@ActiveProfiles("test") //테스트 코드에서 사용할 profile 지정 
class SimpleProductServiceTest {

    @Autowired 
    SimpleProductService  simpleProductService;

    //@Transactional //트랜잭셔널한 처리를 위해 사용 (테스트 코드에 사용할 경우 테스트 후 커밋이 아닌 롤백)
    //데이터베이스 사용 시 주석 해제
    @Test 
    @DisplayName("상품을 추가한 후 id로 조회하면 해당 상품이 조회되어야 한다.") //테스트 코드1
    void productAddAndFindByIdTest (){
        ProductDto productDto = new ProductDto("연필", 300, 20); //테스트이기에 생성자가 아닌 필드에 바로 주입

        ProductDto savedProductDto = simpleProductService.add(productDto);
        Long savedProductId = savedProductDto.getId();

        ProductDto foundProductDto = simpleProductService.findById(savedProductId);

        //assertTrue 메소드로 성공 및 실패 여부 자동으로 확인 
        //동등성 비교를 위해 equals 메소드 사용
        //데이터베이스의 findById는 새로운 Product 인스턴스를 생성하는 구조이기에 동일성 비교 시(==) 테스트 실패
        assertTrue(savedProductDto.getId().equals(foundProductDto.getId()));
        assertTrue(savedProductDto.getName().equals(foundProductDto.getName()));
        assertTrue(savedProductDto.getPrice().equals(foundProductDto.getPrice()));
        assertTrue(savedProductDto.getAmount().equals(foundProductDto.getAmount()));
    }

    @Test
    @DisplayName("존재하지 않는 상품 id로 조회하면 EntityNotFoundException이 발생해야한다.") //테스트 코드2
    void findProductNotExistIdTest(){
        Long notExistId = -1L;

        //assertThrows 메소드로 예외발생 테스트 
        //assertThrows 메소드의 인자 1. 어떤 예외가 발생해야하는지 2. 예외가 발생해야할 코드 
        assertThrows(EntityNotFoundException.class, ()->{
           simpleProductService.findById(notExistId);
        });
    }
}
```

  -> 위의 경우 특정 메소드를 테스트하기에 통합 테스트로 적절하지 않음. 즉, 단위 테스트로 구현하는 것이 적절


- 단위테스트: add 작동 테스트

```ruby
@ExtendWith(MockitoExtension.class) //모킹을 사용하여 단위테스트 (스프링부트 실행x)
public class SimpleProductServiceUnitTest {
    @Mock //해당 의존성에 목 객체 주입 (주입만으로는 아무 기능 x)
    private ProductRepository productRepository;
    @Mock //모킹을 하지 않더라도 목 객체 주입해야함.
    private ValidationService validationService;

    @InjectMocks //@Mock으로 주입한 목 객체들을 simpleProductService 내에 있는 의존성에 주입
    private  SimpleProductService simpleProductService; //실제 인스턴스 생성

    @Test
    @DisplayName("상품 추가 후에는 추가된 상품이 반환되어야한다") 
    void productAddTest (){
        ProductDto productDto = new ProductDto("연필", 300, 20);
        Long PRODUCT_ID = 1L;

        Product product = ProductDto.toEntity(productDto);
        product.setId(PRODUCT_ID);

        //when 메소드로 목 객체의 동작 정의 
        //when(A).thenReturn(B) -> 목 객체가 A에 해당하는 동작을 수행할 때 B에 있는 값을 반환
        when(productRepository.add(any())).thenReturn(product);

        ProductDto savedProductDto = simpleProductService.add(productDto);

        assertTrue(savedProductDto.getId().equals(PRODUCT_ID));
        assertTrue(savedProductDto.getName().equals(productDto.getName()));
        assertTrue(savedProductDto.getPrice().equals(productDto.getPrice()));
        assertTrue(savedProductDto.getAmount().equals(productDto.getAmount()));
    }

}
```
