# Vanilla JS로 스타크래프트 구현하기 (1) - Canvas API를 활용하여 맵과 유닛을 그리고 유닛 선택하기

## 들어가며

정말 오랜만의 포스팅입니다. 그래서 근황 얘기를 하자면, 올해 5월부터 [줌인터넷](https://www.zuminternet.com) 에서 프론트엔드 개발자로 일하고 있습니다. 대표적으로 [줌닷컴](https://zum.com/) 이라는 포탈사이트를 운영하는 회사입니다.  저희 개발 팀에 대해 더 궁금하시다면 [줌인터넷 기술블로그](https://zuminternet.github.io/) 를 통해 더 자세히 아실 수 있습니다.

그렇게 일하고 있는데, 업무 외에 지적 자극이 되는 토이 프로젝트를 하고 싶었습니다. 이래저래 생각해본 결과 **스타크래프트**를 프레임워크없이 구현해보는 것은 어떨까 하는 생각이 들었습니다.

![share-1200x630 ff395bac028b629b277bec2f63bf3632bd8de8bd](https://user-images.githubusercontent.com/41279460/129761485-93a14013-ae23-450d-a452-de03a3754098.jpg)

스타크래프트, 아시죠? 한국의 ~~민속놀이~~

크게 세 가지 이유로 스타크래프트를 주제로 택했습니다.

1. 평소에 게임을 좋아하고 스타크래프트를 즐겨합니다. 재밌게 진행할 수 있으리라 생각했습니다.
2. 객체지향 프로그래밍과 구조 설계를 연습하기 좋다고 생각했습니다. 시스템, 유닛, 건물, 자원 등 오브젝트들로 가득 차 있습니다. 그러한 오브젝트들이 효과적으로 통신하도록 구조를 설계해야 합니다.
3. 구현량이 많을 것으로 예상됩니다. 자바스크립트를 마음껏 사용하고 공부해볼 수 있으리라 생각 했습니다.

**프레임워크, 라이브러리는 사용하지 않고**, 요즘은 OLOO 패턴에 꽂혔으므로 **OLOO(Objects Linked to Other Objects) 패턴을 사용**하여 진행합니다. 그리고 시리즈로 장기 연재할 예정입니다.

**작업 내용은 [개인 저장소](https://github.com/elrion018/js-craft)에서 확인해보실 수 있습니다.**

## 우선 맵을 만들어보자

일단 무언가 존재할 맵이 있어야 합니다. 이 맵 위에 유닛 등 오브젝트를 그릴 것입니다. [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)를 활용하여 봅시다. 

```jsx
export const field = {
  init: function (app) {
    this.setCanvasAndContext(app);
    requestAnimationFrame(this.updateCanvas.bind(this));
  },

	// canvas element를 생성, context를 설정하고 추가하는 메소드
  setCanvasAndContext: function (app) {
    this.canvas = document.createElement('canvas');
    this.ctx = this.canvas.getContext('2d');
    this.canvas.width = window.innerWidth;
    this.canvas.height = window.innerHeight;

    app.appendChild(this.canvas);
  },

	// canvas의 상태를 갱신하는 메소드
  updateCanvas: function () {
    if (this.mouseIsPressed && this.endX && this.endY) {
      this.drawDragRect();
    }

    requestAnimationFrame(this.updateCanvas.bind(this));
  },

	// 마우스 interaction으로 발생하는 event들에 handler들을 붙여주는 메소드
  addListenersForMouseEvent: function () {
    this.canvas.addEventListener('mousemove', this.handleMouseMove.bind(this));
    this.canvas.addEventListener('mousedown', this.handleMouseDown.bind(this));
    this.canvas.addEventListener('mouseup', this.handleMouseUp.bind(this));
  },

  handleMouseUp: function (event) {
    this.mouseIsPressed = false;
    this.endX = null;
    this.endY = null;
  },

  handleMouseDown: function (event) {
    const { offsetX, offsetY } = event;
    this.startX = offsetX;
    this.startY = offsetY;

    this.mouseIsPressed = true;
  },

  handleMouseMove: function (event) {
    if (this.mouseIsPressed) {
      const { offsetX, offsetY } = event;
      this.endX = offsetX;
      this.endY = offsetY;

      this.drawDragRect();
    }
  },

	// 마우스 드래그 시 녹색 사각형을 생성하는 메소드
  drawDragRect: function () {
    this.ctx.strokeStyle = 'green'; // 선 색깔은 그린
    this.ctx.lineWidth = 1;

    this.ctx.beginPath();
    this.ctx.rect(
      this.startX,
      this.startY,
      this.endX - this.startX,
      this.endY - this.startY
    );
    this.ctx.stroke();
  },
};
```

`canvas element` 를 생성하여 추가해줍시다. 일단 `viewport` 만큼 캔버스의 높이와 넓이를 설정했습니다. 그리고 캔버스의 상태를 갱신해주는 메소드(`updateCanvas`)를 추가해줍시다. 이 메소드 내엔 앞으로 갱신해주고 싶은 대상을 얼마든지 추가해주어도 됩니다. 

(유닛이라든지) 이 메소드는 `requestAnimationFrame`(다음 리페인트 전에 해당 메소드를 시작해달라는 의미인데, 추후 정리하겠습니다. ) 을 통해 재귀적으로 실행해줍시다. 그래야 계속 상태변화를 추적할 수 있습니다.  그리고 하는 김에 유닛 선택 시 사용하는 드래그 사각형도 그려봅시다.

![green-drag](https://user-images.githubusercontent.com/41279460/129761565-150d1841-598e-4f6f-8e29-e798a82afcca.gif)

녹색 사각형이 예쁘게 그려지고 있죠?

## 이제 유닛을 추가하고 드래그로 선택해보자

이제 유닛을 추가해봅시다. 처음부터 복잡할 필요는 없습니다. 작은 동그라미 정도면 충분합니다.

```jsx
export const unit = {
  init: function (positionX, positionY) {
    this.positionX = positionX;
    this.positionY = positionY;
    this.radius = 20;
    this.isSelected = false;
  },

  getPositions: function () {
    return {
      positionX: this.positionX,
      positionY: this.positionY,
    };
  },

  setIsSelected: function (isSelected) {
    this.isSelected = isSelected;
  },
};
```

`positionX` 와 `positionY` 로 유닛의 위치를, `radius` 로 유닛의 크기를 나타내봅시다. 그리고 `isSelected`는 유닛의 선택 여부를 나타냅니다.

```jsx
import { unit } from './unit.js';

export const system = {
  init: function () {
    this.units = [];
  },

  createUnit: function (positionX, positionY) {
    const createdUnit = Object.create(unit);
    this.matrix[positionY][positionX] = 1;

    createdUnit.init(positionX, positionY);
    this.units.push(createdUnit);
  },

  getUnits: function () {
    return this.units;
  },

  selectUnitsWithDrag: function (startX, startY, endX, endY) {
    const leftX = Math.min(startX, endX);
    const rightX = Math.max(startX, endX);
    const topY = Math.max(startY, endY);
    const bottomY = Math.min(startY, endY);

    this.units.forEach(unit => {
      const { positionX, positionY } = unit.getPositions();

      if (
        positionX >= leftX &&
        positionX <= rightX &&
        positionY <= topY &&
        positionY >= bottomY
      ) {
        unit.setIsSelected(true);

        return;
      }

      unit.setIsSelected(false);
    });
  },
};
```

전체 시스템을 관리하는 객체를 만들고 `unit` 객체를 프로토타입으로 삼는 객체를 `units` 배열에 추가해주는 `createUnit` 메소드를 추가해줍시다. 그리고 `selectUnitsWithDrag` 메소드는 드래그 사각형의 영역에 유닛의 `positionX` 와 `positionY` 의 동시 포함 여부에 따라 유닛의 `isSelected` 에 변화를 줍니다.

```jsx
export const field = {
  init: function (app, gameSystem) {
    this.gameSystem = gameSystem;
    this.endX = null;
    this.endY = null;

    this.setCanvasAndContext(app);
    this.addListenersForMouseEvent();

    requestAnimationFrame(this.updateCanvas.bind(this));
  },

  setCanvasAndContext: function (app) {
    this.canvas = document.createElement('canvas');
    this.ctx = this.canvas.getContext('2d');
    this.canvas.width = window.innerWidth;
    this.canvas.height = window.innerHeight;

    app.appendChild(this.canvas);
  },

  updateCanvas: function () {
    if (this.mouseIsPressed && this.endX && this.endY) this.drawDragRect();

    if (this.gameSystem.units.length) this.drawUnits();

    requestAnimationFrame(this.updateCanvas.bind(this));
  },

  addListenersForMouseEvent: function () {
    this.canvas.addEventListener('mousemove', this.handleMouseMove.bind(this));
    this.canvas.addEventListener('mousedown', this.handleMouseDown.bind(this));
    this.canvas.addEventListener('mouseup', this.handleMouseUp.bind(this));
  },

  handleMouseUp: function (event) {
    this.mouseIsPressed = false;
    if (this.isDraged) {
      this.gameSystem.selectUnitsWithDrag(
        this.startX,
        this.startY,
        this.endX,
        this.endY
      );
      this.isDraged = false;
    }

    this.endX = null;
    this.endY = null;
  },

  handleMouseDown: function (event) {
    const { offsetX, offsetY } = event;
    this.startX = offsetX;
    this.startY = offsetY;

    this.mouseIsPressed = true;
  },

  handleMouseMove: function (event) {
    if (this.mouseIsPressed) {
      const { offsetX, offsetY } = event;
      this.endX = offsetX;
      this.endY = offsetY;
      this.isDraged = true;

      this.drawDragRect();
    }
  },

  drawDragRect: function () {
    this.ctx.strokeStyle = 'green'; // 선 색깔은 그린
    this.ctx.lineWidth = 1;

    this.ctx.beginPath();
    this.ctx.rect(
      this.startX,
      this.startY,
      this.endX - this.startX,
      this.endY - this.startY
    );
    this.ctx.stroke();
    this.ctx.strokeStyle = 'black';
  },

  drawUnits: function () {
    this.gameSystem.getUnits().forEach(unit => {
      const { positionX, positionY, radius } = unit;
      this.ctx.beginPath();
      this.ctx.arc(positionX, positionY, radius, 0, 2 * Math.PI);

      if (unit.isSelected) this.ctx.fill();
      this.ctx.stroke();
    }, this);
  },
};
```

`gameSystem` 을 주입받게 되었습니다. `units` 배열에 접근하기 위해서입니다. `updateCanvas` 에서 `drawUnits` 메소드를 지속 호출하면서 캔버스 위에 유닛을 그리고 있습니다. 

유닛이 선택되었을 때(`isSelected`가 `true` 일 때) 유닛이 검은 색으로 채워지게 되는데 이 또한 그려낼 것입니다. 

`drawDragRect` 메소드로 드래그 삼각형을 그리고 있고, 마우스 버튼에서 손가락을 땠을 때(`mouseup`) `selectUnitsWithDrag` 메소드를 호출하면서 `startX` , `startY` , `endX`, `endY` 를 기준으로 유닛들을 선택하고 있습니다.

![green-drag-select](https://user-images.githubusercontent.com/41279460/129761605-3881725b-714e-4fa6-ae4e-9c97c8f9d0a2.gif)

이제 유닛을 선택할 수 있습니다!

## 마치며

오늘은 맵과 유닛을 만들고 선택해보았습니다. 하지만, 멈춰있는 유닛은 쓸모가 없겠죠. 다음 포스팅엔 유닛을 움직이게 만들어 보겠습니다. 단순히 직선거리만 고려하여 이동하는 간단한 형태와 더 나아가 장애물까지 고려하여 우회하면서 최단거리로 이동하는 형태가 될 것 같습니다.

질문 사항이나 피드백은 편하게 댓글 남겨주세요. 감사합니다!
