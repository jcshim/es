# 프람프트
``` 
Arduino -> 시리얼 -> PC: processing Program
스위치 on -> 화면 붉은색
스위치 off -> 화면 푸른식

1. 회로만들기 INPUT_PULLUP
2. 아두이노 코드 작성 및 Upload
3. 프로세싱 서버 만들기, 200 X 200
 
```

바로 해볼 수 있게 회로 설명 + 아두이노 코드 + 프로세싱 코드 한 번에 드릴게요 😊

---

## 1. 회로 만들기 (INPUT_PULLUP 이용)

스위치 1개, 아두이노 UNO 기준으로 설명하겠습니다.

* 사용핀: **디지털 2번 핀**
* 내부 풀업 저항 사용 → `INPUT_PULLUP`

**배선**

* 스위치 한쪽 → **아두이노 D2 핀**
* 스위치 다른쪽 → **GND**

이렇게 하면:

* 스위치 **안 눌림 (OFF)**: D2 = HIGH (5V 쪽으로 풀업)
* 스위치 **눌림 (ON)**: D2 = LOW (GND로 연결)

우리는 **스위치 눌림(ON) → 빨간 화면**, **뗌(OFF) → 파란 화면**으로 처리하겠습니다.

---

## 2. 아두이노 코드 (스위치 상태를 시리얼로 전송)

* 보드: Arduino Uno 가정
* Baudrate: **9600**
* 스위치 상태가 변할 때만 `"1"` 또는 `"0"`를 전송

```cpp
// Arduino 코드: 스위치 상태를 시리얼로 PC(Processing)로 전송
const int SWITCH_PIN = 2;  // 스위치 입력 핀

int lastState = HIGH;  // INPUT_PULLUP이므로 기본은 HIGH (스위치 OFF 상태)

void setup() {
  pinMode(SWITCH_PIN, INPUT_PULLUP); // 내부 풀업 사용
  Serial.begin(9600);                // PC와 시리얼 통신
  delay(1000);                       // 포트 안정화 약간 기다림

  // 초기 상태 한 번 보내기
  int state = digitalRead(SWITCH_PIN);
  if (state == LOW) {
    Serial.println("1");  // ON
  } else {
    Serial.println("0");  // OFF
  }
}

void loop() {
  int state = digitalRead(SWITCH_PIN);

  // 상태가 변했을 때만 PC로 전송 (노이즈/트래픽 줄이기)
  if (state != lastState) {
    lastState = state;

    if (state == LOW) {
      // 스위치 ON (눌림) -> 1 전송
      Serial.println("1");
    } else {
      // 스위치 OFF (안 눌림) -> 0 전송
      Serial.println("0");
    }

    delay(10);  // 아주 간단한 디바운스용 딜레이
  }
}
```

이 코드 업로드 후, **툴 → 시리얼 모니터**에서 속도 9600으로 맞추고 스위치를 눌러보면
`1` / `0` 이 찍히는지 먼저 확인해 보세요. (문자 + 줄바꿈 전송)

---

## 3. Processing 코드 (COM7에서 읽어서 화면 색 바꾸기)

* 윈도우 기준, 시리얼 포트는 **"COM7"** 사용 (문제에서 가정한 값)
* 윈도우 크기: **200 x 200**
* `"1"` 읽으면 빨강, `"0"` 읽으면 파랑

```java
// Processing 코드 (Processing 3.x 기준)
// PC에서 Arduino가 보내는 1/0을 읽어서
// 1이면 빨간색, 0이면 파란색으로 배경을 바꾼다.

import processing.serial.*;

Serial myPort;
color currentColor;

void setup() {
  size(200, 200);  // 윈도우 크기
  // COM7 포트, Baudrate 9600으로 오픈
  // (윈도우 장치 관리자에서 실제 포트 번호 확인 필요)
  String portName = "COM7";
  myPort = new Serial(this, portName, 9600);

  // 줄 단위로 읽기 위해 개행문자 나올 때까지 버퍼
  myPort.bufferUntil('\n');

  // 초기 배경 색: 스위치 OFF 가정 → 파란색
  currentColor = color(0, 0, 255);
}

void draw() {
  background(currentColor);
}

// 시리얼로 '\n'까지 데이터가 들어오면 자동 호출됨
void serialEvent(Serial p) {
  String inString = p.readStringUntil('\n');
  if (inString != null) {
    inString = trim(inString);  // \r, \n, 공백 제거

    // "1" = 스위치 ON → 빨간색
    if (inString.equals("1")) {
      currentColor = color(255, 0, 0);
    }
    // "0" = 스위치 OFF → 파란색
    else if (inString.equals("0")) {
      currentColor = color(0, 0, 255);
    }

    // 디버깅용 출력 (콘솔에 표시)
    println("Received: " + inString);
  }
}
```

---

## 실행 순서 요약

1. **회로 연결**

   * D2 ↔ 스위치 ↔ GND
   * 아두이노 USB로 PC 연결

2. **아두이노 코드 업로드**

   * 보드/포트 설정 → 업로드
   * 시리얼 모니터에서 `1`/`0` 변하는지 테스트해봐도 좋음

3. **Processing 실행**

   * 위 Processing 코드 복사
   * `Run` 버튼 실행
   * 창(200x200) 뜨면 스위치 눌러보기

     * 스위치 ON(눌림) → 빨간색
     * 스위치 OFF(안 눌림) → 파란색

필요하면 이걸 과제용으로 단계별 실습 설명(보고서용 텍스트)까지 정리해 드릴게요.
