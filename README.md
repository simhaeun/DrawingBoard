# DrawingBoard
<img src="https://img.shields.io/badge/Parcel-9E66BF?style=for-the-badge&logo=Parcel&logoColor=white"> <img src="https://img.shields.io/badge/Sass-CC6699?style=for-the-badge&logo=Sass&logoColor=white"> <img src="https://img.shields.io/badge/ESLint-4B32C3?style=for-the-badge&logo=ESLint&logoColor=white"> <img src="https://img.shields.io/badge/Prettier-F7B93E?style=for-the-badge&logo=Prettier&logoColor=white"> <img src="https://img.shields.io/badge/FontAwesome-528DD7?style=for-the-badge&logo=FontAwesome&logoColor=white">

![image](https://user-images.githubusercontent.com/58839497/217749092-a3fe11e1-4888-4710-a730-c36a4241ad94.png)
https://hacookie-drawingboard.netlify.app/

### 번들러 
- Parcel (https://ko.parceljs.org/)
- 구성 없는 단순한 자동 번들링
- 소/중형 프로젝트에 적합
- webpack과 다르게 별다른 설정없이 사용이 가능하다.

### canvas를 활용한 그림판 구현
```jsx
initContext(){
  this.context = this.canvasEl.getContext("2d");
}
```
`<canvas>` 요소는 `getContext()` 메서드를 이용해서, 랜더링 컨텍스트와 그리기 함수들을 사용할 수 있습니다. getContext() 메서드는 렌더링 컨텍스트 타입을 지정하는 하나의 파라메터를 가집니다. 본 프로젝트에서 다루고 있는 2D 그래픽의 경우, CanvasRenderingContext2D (en-US)을 얻기위해 "2d"로 지정합니다.

### 3가지 MODE
- 아무것도 아닌 상태 (NONE)
- 그리기 (BRUSH)
- 지우기 (ERASER)

<br/>

## BRUSH MODE

#### BRUSH 아이콘을 누르면 마우스 커서와, MODE 변경
```jsx
onClickBrush(event) {
  const IsActive = event.currentTarget.classList.contains("active");
  this.MODE = IsActive ? "NONE": "BRUSH";
  this.canvasEl.style.cursor = IsActive ? "default" : "crosshair";
  this.brushEl.classList.toggle("active");
}
```

#### 마우스 좌표값 구하기
```jsx
getMousePosition(event) {
  const boundaries = this.canvasEl.getBoundingClientRect();
  return {
    x: event.clientX - boundaries.left,
    y: event.clientY - boundaries.top,
  };
}
```

#### 선 그리기
```jsx
IsMouseDown = false;

onMouseDown(event) {
  if(this.MODE === "NONE") return;
  this.IsMouseDown = true;
  const currentPosition = this.getMousePosition(event);
  this.context.beginPath(); // 경로 시작
  this.context.moveTo(currentPosition.x, currentPosition.y);
  this.context.lineCap = "round";
  this.context.strokeStyle = "#000000";
  this.context.lineWidth = 10;
}
onMouseMove(event) {
  if(!this.IsMouseDown) return;
  const currentPosition = this.getMousePosition(event);
  this.context.lineTo(currentPosition.x, currentPosition.y); // 움직여줘
  this.context.stroke(); // 그려줘
}
onMouseUp(event) {
  if(this.MODE === "NONE") return;
  this.IsMouseDown = false;
}
```

#### BRUSH 색상 선택
```html
<input class="tool colorSelector" id="colorPicker" type="color" value="#e53935">
```
- `<input type="color">` : 색을 선택할 수 있는 입력 필드(color picker)

```jsx
assingElement() {
  this.colorPickerEl = this.toolbarEl.querySelector("#colorPicker");
}
onMouseDown(event) {
  // this.context.strokeStyle = "#000000";
  this.context.strokeStyle = this.colorPickerEl.value; // input 값 가져오기
}
```

#### BRUSH 사이즈 조절
```html
<div class="brushPanel hide" id="brushPanel">
  <label for="brushSize" class="brushSizeLabel">Size</label>
  <input type="range" class="brushSize" id="brushSize" value="3" min="1" max="80">
</div>
```
```jsx
assingElement() {
  this.brushPanelEl = this.containerEl.querySelector("#brushPanel");
  this.brushSliderEl = this.brushPanelEl.querySelector("#brushSize");
}
addEvent() {
  this.brushSliderEl.addEventListener("input", this.onChangeBrushSize.bind(this));
}
onMouseDown(event) {
  // this.context.lineWidth = 10;
  this.context.lineWidth = this.brushSliderEl.value;
}
```

<br/>

## ERASER MODE
```jsx
eraserColor = "#FFFFFF"

onMouseDown(event) {
  if(this.MODE === "BRUSH") {
    this.context.strokeStyle = this.colorPickerEl.value;
    this.context.lineWidth = this.brushSliderEl.value;
  } else if(this.MODE === "ERASER") {
    this.context.strokeStyle = this.eraserColor;
    this.context.lineWidth = 50;
  }
}
```

#### 캔버스 바탕색 초기화 (투명 -> #FFF)
```jsx
backgroundColor = "#FFFFFF"

constructor() {
  this.initCanvasBackgroundColor();
}

initCanvasBackgroundColor() {
  this.context.fillStyle = this.backgroundColor;
  this.context.fillRect(0, 0, this.canvasEl.width, this.canvasEl.height);
}
```

<br/>

## 실행 취소
#### 실행 취소 하기 전 상태 저장
```jsx
undoArray = [];

savaState() {
  this.undoArray.push(this.canvasEl.toDataURL()); // 현재 캔버스 상태를 dataURL 컨버팅
}
```
- `toDataURL` : 캔버스의 데이터 URL을 가져올 수 있습니다. Canvas에 그린 그림을 이미지로 만들기

#### 실행 취소
이미지가 로드 된 시점에 캔버스를 지우고, 저장한 이미지를 캔버스에 그려내는 방식
```jsx
onClickUndo() {
  if(this.undoArray.length === 0) {
    alert("더이상 실행취소 불가합니다!");
    return
  };
  let previousDataUrl = this.undoArray.pop(); // 가장 최근 것 꺼내오기
  let previousImage = new Image(); // 이미지로 로드

  previousImage.onload = () => {
    this.context.clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
    this.context.drawImage(previousImage, 0, 0, this.canvasEl.width, this.canvasEl.height, 0, 0, this.canvasEl.width, this.canvasEl.height);
  }
  
  previousImage.src = previousDataUrl;
}
```
- `onload` : 문서의 모든 콘텐츠(images, script, css, etc)가 로드된 후 발생하는 이벤트
- `clearRect()` : 화면을 지우는 메소드
- `drawImage()` : 캔버스에 이미지를 그리는 다양한 방법을 제공

#### 최대 5번까지 실행 취소 가능
```jsx
savaState(){
  if (this.undoArray.length > 4) {
    this.undoArray.shift();
    this.undoArray.push(this.canvasEl.toDataURL());
  } else {
    this.undoArray.push(this.canvasEl.toDataURL());
  }
}
```

<br/>

## 초기화
```jsx
onClickClear() {
  this.context.clearRect(0, 0, this.canvasEl.width, this.canvasEl.height);
  this.undoArray = [];
  this.initCanvasBackgroundColor(); 
}
```
<br/>

## 이미지 다운로드
다운로드 받을 이미지의 형식과 이름 지정
```html
<a id="download"><i class="fas fa-download"></i></a>
```
```jsx
assingElement() {
  this.downloadLinkEl = this.toolbarEl.querySelector("#download");
}
onClickDownload() {
  this.downloadLinkEl.href = this.canvasEl.toDataURL("image/jpeg", 1);
  this.downloadLinkEl.download = "exmaple.jpeg";
}
```
