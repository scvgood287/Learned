## 개요
같이 공부를 하는 Kyusang Jung 님의 20210331_Counting Card Variation.md를 보고 영감을 받아
BlackJack을 구현해보고자 하게되었다.

출처 - https://github.com/Carrymachine/FE-Session-Retrospective/blob/main/20210331_Counting%20Card%20Variation.md

### 구현

코드 -  https://codepen.io/scvgood287/pen/YzNNwWm?editors=0011

#### 블랙잭의 기본적인 룰을 콘솔 게임으로 구현

1. gameStart()로 게임을 시작.
2. setBetAmount()로 배팅금액을 설정.
3. 배팅 금액이 설정되면 자동으로 카드 배부, 그 후 딜러와 플레이어의 블랙잭 체크, 블랙잭이면 게임 종료. 아니면 플레이어의 턴.
4. hit(), doubleDown()로 카드를 더 받을 수 있음. stay()로 턴 종료.  hit이나 doubleDown 도중 bust하면 게임 패배.
5. 턴을 종료하면 딜러의 턴 시작, 딜러는 자신의 패의 합이 16보다 작으면 카드를 계속 받음, 17이 넘으면 딜러의 턴 종료. bust하면 게임 승리.
6. 게임의 승패를 확인, doubleDown의 경우 두 배의 금액을 얻음.

#### 세부적으로 신경 쓴 부분

- gameState
게임의 페이즈에 따라 사용가능한 함수들을 제어하기 위함. 0부터 시작. 게임이 진행됨에 따라 증가. 0 = 게임 시작 전, 1 = 베팅 금액 정하기, 2 = 플레이어 턴
예) gameState 가 1일때는 hit, doubleDown, stay를 사용할 수 없음.

- checkResult
한 세트를 초기화 하기위해 gameEnd()함수를 게임이 종료할 때 호출하는데, 매번 승패를 확인하는 코드와 console.log()를 마지막 줄에 넣기보다는,
checkResult만 바꿔 게임의 승패를 정해둔 상태로 gameEnd()를 실행하는게 더 좋을거 같아서 만듬.
지금 생각해보니 gameEnd(checkResult) 형식이 나았던거 같음.

- checkDoubleDown, canUseHitOrDoubleDown
checkDoubleDown 
더블다운을 했는가 체크. 더블다운으로 게임을 이기거나 졌을때 특별한 메시지를 보여주기 위해 만든 것.
canUseHitOrDoubleDown 
카드를 추가로 받을 수 있는가에 대한 체크. setBetAmount로 배팅액을 설정하여 처음 카드를 받은 후 남은 카드가 없다거나, hit 도중 덱에 남은 카드가 없을때
더 이상 hit나 doubleDown을 사용하지 못하게 하기 위한 것.

#### 아쉬운 부분들

- playerUpdateHands(), dealerUpdateHands()
나름 생각을 하여 만들었으나, Ace가 두개 들어왔을 경우를 상정하지 못함.

- 전체적으로 js를 제대로 사용하지 못한 점과, 객체지향적 코딩 습관이 들여져 있지 않았음. 남에게 설명하기 어려운 코드(내가 한 달 후에 봐도 설명 못할 코드, 테스트가 어려운 코드.)

- 마치 자연어로 말을 쭉 늘여 써놓은 것뿐인 단순한 코드였다.

### Hun Kang 선생님의 코드 리뷰

#### 피드백

1. 게임의 진행 상태, 결과를 Global state 로 관리하려는 접근은 좋았다
2. 딜러, 플레이어 턴에 실행해야 하는 행동을 하나의 함수에서 while, if, 등의 분기/반복문을 사용해 복잡도가 높아졌다
3. 각각의 행동(분기) 들을 작은 함수 단위로 분리하고, 여러 함수의 묶음으로 deal 과정을 표현할 수 있게 리팩토링해봤다 (Dealer 한정)
4. Dealer, Player 각자 실행할 수 있는 행동(메서드)이 다르므로, 클래스와 같은 객제지향 기능을 사용해 변수와 함수를 묶어 표현하는 것이 더 가독성이 좋다

#### 문제점을 어느정도 개선하여 딜러부분만 리팩토링한 코드

```js
class Dealer {
  constructor(limit, deck) {
    this.limit = limit;
    this.deck = deck;
    this.total = 0;
  } 
  getCardUntil(globalDeck) {
    while(this.isLessThan(this.limit)) {
      this.pullCardFrom(globalDeck);
      this.setTotal();
    }
  }
  setTotal() {
    this.totalNumber = 0;
    const aces = this.deck.filter((card) => card.mark === 'A');
    const exceptAce = this.deck.filter((card) => card.mark !== 'A');
    
    [...exceptAce].forEach((card) => {
      this.total = getNumberFrom(card.mark, this.total);
    });
    if (aces.length === 1) {
      this.total <= 10 ? this.total += 11 : this.total += 1;
    }
    if (aces.length > 1) {
      const remainingAce = aces.length - 1;
      
      this.total + remainingAce <= 10 ?
        this.total + remainingAce + 11 :
        this.total + aces.length;
    }
  }
  isLessThan(number) {
    this.setTotal();
    if (this.total < number) return new Error('E_DEALER_MIN_NUMBER');
  }
  pullCardFrom(globalDeck) {
    dealer.deck.push(globalDeck.pop());
  }
  cast(globalDeck) {
    this.getCardUntil(globalDeck);
    if (isBusted(this.total)) {
      return 'dealer Busted!'
    }
  }
}
// -copyright Hun Kang all rights reserved.
```

### 목표

이번 리뷰를 통해 알게 된 점을 정리해 공부 한 뒤 자신의 전체 코드를 리팩토링 할 예정.
