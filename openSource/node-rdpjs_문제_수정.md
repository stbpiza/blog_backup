# [오픈소스] node-rdpjs 문제 수정

![](https://velog.velcdn.com/images/stbpiza/post/9eb9c362-ff95-4193-9def-2b95c173a443/image.png)

# node-rdpjs 문제 수정
## 소개
```
💡 Node.js에서 RDP 접속을 할 수 있도록 연결해주는 라이브러리
```
https://github.com/stbpiza/node-rdpjs

## 기여

기존 제작자들이 업데이트를 오랜기간 멈춘 상황
포크해서 오류 수정 후 풀 리퀘스트는 보내지 않고 바로 사용했습니다.

## 문제점
RDP 접속을 해서 윈도우를 사용 중 마우스 휠이 오작동하는 문제 발견
아래쪽 방향으로는 휠이 잘 동작하나, 위쪽 방향으로 돌리면 올라갔다 내려갔다 하는 문제 발생

## 코드 분석

/node-rdpjs/lib/protocol/rdp.js
```javascript
/**
 * Wheel mouse event
 * @param x {integer} mouse x position
 * @param y {integer} mouse y position
 * @param step {integer} wheel step
 * @param isNegative {boolean}
 * @param isHorizontal {boolean}
 */
RdpClient.prototype.sendWheelEvent = function (x, y, step, isNegative, isHorizontal) {
	if (!this.connected)
		return;
	
	var event = pdu.data.pointerEvent();
	if (isHorizontal) {
		event.obj.pointerFlags.value |= pdu.data.PointerFlag.PTRFLAGS_HWHEEL;
	}
    else {
		event.obj.pointerFlags.value |= pdu.data.PointerFlag.PTRFLAGS_WHEEL;
    }
	console.log('event1 ' + JSON.stringify(event.obj.pointerFlags.value));
        
	if (isNegative) {
		event.obj.pointerFlags.value |= pdu.data.PointerFlag.PTRFLAGS_WHEEL_NEGATIVE;
	}
	console.log('event2 ' + JSON.stringify(event.obj.pointerFlags.value));
        
    event.obj.pointerFlags.value |= (step & pdu.data.PointerFlag.WheelRotationMask)
	
    event.obj.xPos.value = x;
    event.obj.yPos.value = y;
    console.log('event3 ' + JSON.stringify(event.obj.pointerFlags.value) + isNegative);

    this.global.sendInputEvents([event]);
}
```
위 코드는 마우스 휠 입력 함수
콘솔로그를 작성해서 코드 실행
휠을 아래로 한칸, 위로 한칸 이동하여 결과 확인

코드 실행결과
```javascript
// 휠 아래로
{"x":790,"y":631,"step":8,"isNegative":true,"isHorizontal":false}
event1 512
event2 768
event3 776true
// 휠 위로
{"x":790,"y":631,"step":8,"isNegative":false,"isHorizontal":false}
event1 512
event2 512
event3 520false
```

코드 실행결과를 분석해보니

1. 아래로 내려가는 경우는 768을 기준으로 그 이상 숫자가 커진만큼 아래로 내려감
2. 위로 올라가는 경우는 512를 기준으로 그 이상 숫자가 커진만큼 위로 올라감

문제는 휠을 조금만 세게 돌려도 +300~400 은 쉽게 커져서 512에서 시작해도 768을 넘어버림
768을 넘으면 올라가는게 아니라 내려가버림
![](https://velog.velcdn.com/images/stbpiza/post/5118bc73-cc13-41a3-9443-3cf2578b4c0f/image.png)

그렇기 때문에 휠을 위로 많이 올리면
768을 못넘는 입력과 넘는 입력이 함께 들어오면서
스크롤이 올라갔다 내려갔다를 반복하는 문제가 발생

## 해결 방법
위로 올라가야하는 경우에 768을 넘으면 767으로 값을 변경해주는 코드를 추가하는 것
```javascript
if (!isNegative && event.obj.pointerFlags.value >= 768) {
		event.obj.pointerFlags.value = 767;
		console.log('event4 ' + JSON.stringify(event.obj.pointerFlags.value) + 'final');
	}
```
수정 결과
```javascript
{"x":801,"y":550,"step":380,"isNegative":false,"isHorizontal":false}
event1 512
event2 512
event3 892false
event4 767final
```
근본적인 해결책은 아니지만
결과적으로 기능은 잘 동작했습니다.

## 사용 방법
기존 라이브러리에 수정한 코드가 적용된 상태는 아니기 때문에,
package.json에 아래와 같이 입력하면 사용 가능합니다.
```JSON
"dependencies": {
    "node-rdpjs": "https://github.com/stbpiza/node-rdpjs",
}
```