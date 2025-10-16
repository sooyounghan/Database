-----
### ERD 완성하기
-----
1. 엔티티, 속성, 식별자, 관계, 카디널리티, 참여도를 하나의 그림으로 표현한 것이 바로 ERD(Entity-Relationship Diagram)다
   - ERD는 복잡한 데이터 구조를 시각적으로 표현해서 개발자, 기획자 등 모든 관계자가 시스템의 청사진을 한눈에 파악하고 소통할 수 있게 도와주는 설계에서 가장 강력한 도구

2. 표기법의 이해와 실무 선택
   - ERD를 그리는 방법에는 대표적으로 '피터 첸 표기법'과 '까마귀발 표기법'이 존재
   - 피터 첸(Peter Chen) 표기법
<div align="center">
<img src="https://github.com/user-attachments/assets/6c4a9181-a8b0-48fd-9bc7-f9964824cbce">
</div>

   - 피터 첸 표기법은 엔티티는 사각형, 관계는 마름모, 속성은 타원으로 표현하는 전통적인 방식
   - 이런 개념을 처음 만든 표기법이라 학술적으로 의미가 있지만, 실무에서는 사용하지 않음
   - 그림이 복잡해지고 공간을 많이 차지
   - 예제에서 회원과 주문만 간단히 표현했는데도 너무 많은 공간을 차지

   - 까마귀발(Crow's Foot) 표기법
<div align="center">
<img src="https://github.com/user-attachments/assets/36ddc570-06a4-402a-af69-2ac912ec2581">
</div>

   - 현재 실무에서 가장 널리 사용되는 사실상의 표준

3. 용어 - 사실상 표준(de facto standard)
   - 사실상 표준은 국가나 국제기구가 공식적으로 정한 표준은 아니지만, 시장에서 널리 사용되어 사실상 표준처럼 자리 잡은 기술이나 제품을 의미
   - 'De Facto'는 '사실상'이라는 의미의 라틴어

4. 왜 실무에서는 까마귀발 표기법을 사용할까?
   - 정보 밀도와 가독성 : 관계를 별도의 마름모로 그리지 않고 엔티티를 잇는 선 자체에 기호로 표현하므로, 훨씬 적은 공간에 더 많은 정보를 깔끔하게 표현할 수 있음
   - 직관적인 표현 : 관계의 'Many(다)' 쪽을 나타내는 기호가 까마귀의 발 모양과 닮았다고 해서 이런 이름이 붙었으며, 기호 자체가 관계의 수량(카디널리티)을 직관적으로 보여줌
   - 대부분의 도구 지원 : MySQL Workbench, DBeaver, ERDCloud 등 우리가 사용하는 대부분의 데이터베이스 모델링 도구가 까마귀발 표기법을 기본으로 지원하므로, 이 표기법만 알면 어떤 도구든 쉽게 사용할 수 있음

5. 까마귀발 표기법의 핵심 기호
   - 엔티티는 사각형 상자로 표현
   - 관계는 두 엔티티를 잇는 직선으로 표현
   - 선의 양 끝에 기호를 붙여 카디널리티와 참여도를 동시에 표현
      + | : 1 (One)
      + < : 다 (Many, 까마귀발 모양)
      + 0 : 0 (Zero)
        * 다는 실제로 발가락 3개 모양 (실제 표현) : <img width="46" height="41" alt="image" src="https://github.com/user-attachments/assets/e56f4439-61d6-4f9d-b0bf-f6d0011670fa" />

   - 기호들을 조합하여 관계를 표현
   - O| (Zero or One) : 0개 또는 1개 (선택적 1)
     + 실제 표현 : <img width="64" height="42" alt="image" src="https://github.com/user-attachments/assets/02704138-aee0-490e-8490-61db64f0de06" />
   - || (One and Only One) : 반드시 1개 (필수적 1)
     + 실제 표현 : <img width="64" height="32" alt="image" src="https://github.com/user-attachments/assets/82e4c90f-81fe-4001-955c-05c3759fa8ea" />
   - O< (Zero or Many) : 0개 또는 여러 개 (선택적 N)
     + 실제 표현 : <img width="56" height="40" alt="image" src="https://github.com/user-attachments/assets/f4fbca07-a46e-480e-958d-05b29b3bd867" />
   - |< (One or Many) : 1개 또는 여러 개 (필수적 N)
     + 실제 표현 : <img width="57" height="40" alt="image" src="https://github.com/user-attachments/assets/53852b25-9dd3-4889-ac6e-233ef975b919" />

6. 표기법에 따라 '쇼핑몰' 관계를 다시 표현
   - 회원 : 주문
      + 한 명의 회원은 주문을 0개 이상 가질 수 있음( O< ) - 선택적 N
      + 하나의 주문은 반드시 한 명의 회원을 가져야 함( || ) - 필수적 1
      + 최종 관계 : 회원 (||)---(O<) 주문

   - 주문 : 상품 (M:N 관계)
      + M:N 관계는 ERD에서 두 엔티티를 직접 연결하고 양쪽에 < 기호를 표기
      + 하나의 주문에는 상품이 1개 이상 포함되어야 함( |< ) - 필수적 N
      + 하나의 상품은 여러 주문에 포함될 수 있으며, 아직 한 번도 주문되지 않았을 수도 있음( O< ) - 선택적 N
      + 최종 관계 : 주문 (>O)---(|<) 상품
<div align="center">
<img src="https://github.com/user-attachments/assets/0d3aa710-a936-40e7-b7cd-3018f74d581f">
</div>

   - 💡 까마귀발 표기법을 읽을 때 가장 중요한 원칙은 "한 엔티티 쪽 끝에 있는 기호는 반대편 엔티티에 대한 규칙을 설명한다"는 것
   - 회원은 여러 주문을 가질 수 있음
   - 회원은 주문을 하지 않을 수 있음 (선택적 관계)
   - 주문은 반드시 하나의 회원에 포함되어야 함 (필수적 관계)
   - 이 모든 것을 종합하면 '쇼핑몰'의 초기 개념적 모델 ERD가 완성
