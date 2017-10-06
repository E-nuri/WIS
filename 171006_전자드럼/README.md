# 아두이노 전자드럼 만들기

문득 youtube에서 노래를 듣다가 자동추천 목록에 드럼 연주곡이 나와서 듣게됐다.

https://www.youtube.com/watch?v=2qSBMZhPkY4

https://www.youtube.com/watch?v=dfDP_2MEfI4

https://www.youtube.com/watch?v=5fmtkuNjw80

위의 연주를 보고 있으면 나도 드럼 쳐보고 싶다는 생각이 들 정도로 재밌게 연주한다.

그래서 드럼을 배워야겠다고 생각하고 드럼을 알아봤는데 1순위 장애요소가 '소음'이었다.

소음을 내 의지로 조절할 수 있는 전자드럼이라는 녀석을 알게 됐는데 또다른 문제는 '가격'



전자드럼 부품 구성을 보면 정말 뭐 별거없다.

드럼 패드에 전기적 신호가 전해지면 그 신호를 드럼 모듈로 전해주고

전자드럼 모듈은 그 신호를 해석해서 내장된 소리를 보내주는거다.



간단한 이론을 알고나니 직접 만들어봐야겠다는 생각이 들었다.

이론 조사하고 테스트까지 다 하고나니 귀찮아서 만드는걸 접었지만..
미래의 내가 변덕을 부려서 만들려고 할게 뻔하기 때문에 간단하게라도 정리를 해야겠다


## 준비물
- 아두이노
- 아두이노와 pc를 연결 할 usb 케이블
- 피에조(piezo) 센서
- 1M옴 저항
- 브레드보드(빵판)
- 맥북(Mac OS 앱스토어에서 garage band를 다운받으면 된다.)
- 가상 midi 소프트웨어 hairless http://projectgus.github.io/hairless-midiserial/


초간단 & 초스피드 튜토리얼
아래 영상을 보고 따라하면 된다.
https://www.youtube.com/watch?v=Pt0YRJAqFGA

코드는 다음과 같다
```
#include <MIDI.h>
MIDI_CREATE_DEFAULT_INSTANCE();

int kick = 36;
int snare = 38;
int led = 13;

void Kick() {
  MIDI.sendNoteOn(kick, 127, 1);
  digitalWrite(led, HIGH);
  delay(1000/2);
  digitalWrite(led, LOW);
  MIDI.sendNoteOff(kick, 0, 1);
  delay(1000/2);
}

void Snare() {
  MIDI.sendNoteOn(snare, 127, 1);
  digitalWrite(led, HIGH);
  delay(1000/2);
  digitalWrite(led, LOW);
  MIDI.sendNoteOff(snare, 0, 1);
  delay(1000/2);
}

void setup() {
  MIDI.begin(1);
  Serial.begin(115200);
}

void loop() {
  Kick();
  Snare();
}
```

핵심 코드는
```
#include <MIDI.h>
MIDI_CREATE_DEFAULT_INSTANCE();
MIDI.sendNoteOn(param1, param2, param3)
MIDI.sendNoteOff(param1, param2, param3)
```
이다.

#include <MIDI.h>
MIDI_CREATE_DEFAULT_INSTANCE();
위의 두 줄은 보는 그대로 MIDI 헤더를 가져와서 쓸 수 있게 환경을 구성하는 것이다.

MIDI.sendNoteOn()과 MIDI.sendNoteOff()에 들어가는 파라미터는 같으므로 MIDI.sendNoteOn()을 기준으로 메모해야지
param1 : 미디 번호를 뜻한다
미디 번호가 뭐냐고?
http://newt.phys.unsw.edu.au/jw/notes.html
위 링크로 들어가면 MIDI number라고 적힌 부분에 들어가는 숫자들이다.
각 숫자는 피아노 건반의 자리라고 생각하면 되는데 이 건반 자리들이 garage band에서도 똑같이 매핑되어 있다.
그러므로 해당 건반이 눌리면(신호가 들어오면) 그에 맞는 소리를 내주는 것이다.

param2 : 벨로시티(velocity) 값이다. 쉽게말해서 세기를 뜻한다. 드럼을 살살 칠 수도 있고 있는 힘껏 칠 수도 있는 일이다.
그런 속성에 맞춰서 값을 준다고 보면 된다.
if-else를 잘 활용하면 신호가 들어오는 범위를 나누고 그에 맞는 출력을 보냄으로써 실제 드럼과 같게 구현할 수 있겠지

param3 : MIDI 통신 채널이다. 따로 채널을 여러개 구성하는게 아니라면(전자드럼 하나만 사용할거라면) 1로 해두면 된다.

전기 신호는 MIDI.sendNoteOn()이 실행된 후 전송되기 시작하고(소리가 나기 시작하고) MIDI.sendNoteOff()가 실행되면 전송이 끊어진다( 소리가 안난다)

MIDI.sendNoteOn() 함수에 대해서 좀 더 자세한 설명은
http://arduinomidilib.sourceforge.net/a00001.html#a9664e523b35d8b42749e7bc3ba888943
http://newt.phys.unsw.edu.au/jw/notes.html
https://randomflik.blogspot.kr/2016/09/arduinomidi.html

를 참고하자.


## 자..그럼 하드웨어를 구축해보자

부품 배치는 아래 링크처럼 하면 된다.
https://www.arduino.cc/en/Tutorial/Knock

피에조 센서는 신호를 0-1023의 영역으로 나누고 받아들인다.
하지만 MIDI.sendNoteOn() 는 0-127까지만 값을 받아들인다.
그렇기에 값의 범위를 적절하게 바꿔줘야하는데 정말 귀찮다면 피에조 센서를 통해 받아들인 값을 나누기 8하는 방법도 있지만
좀 더 효율적으로 작성하려면 아두이노의 map()함수를 활용하면된다.

```
map(value, fromLow, fromHigh, toLow, toHigh)
value : 신호가 들어오는 곳(변수)
fromLow : 입력되는 신호의 최저범위(피에조는 0~1023이므로 0)
fromHigh : 입력되는 신호의 최고범위(피에조는 0~1023이므로 1023)
toLow : 발송되는 신호의 최저범위(sendNoteOn()함수는 0~127이므로 0)
toHigh : 발송되는 신호의 최고범위(sendNoteOn()함수는 0~127이므로 127)
```

레퍼런스 링크 : https://www.arduino.cc/en/Reference/Map



위의 상태까지 코딩을 하면 정말정말 기본적인 드럼 구성은 끝난다.



## 앞으로 할 것들
- hairless를 이용하는게 아닌 진짜 midi로 구현하기
http://www.instructables.com/id/Homemade-Electronic-Drum-Kit-With-Arduino-Mega2560/
https://github.com/Victor2805/Homemade-electronic-drum-kit-with-arduino/blob/master/Code.ino
https://www.youtube.com/watch?v=rmfAqg9O_os

- 실제 시제품처럼 드럼의 구조 구현
  예를들면 오픈하이햇, 클로즈 하이햇, 듀얼트리거(스네어 패드에서 림샷을 연주하는 것처럼 듀얼 트리거 구현)
  https://dtinth.github.io/midi-instruments/#drums
  JSConf2016 참가했을 때 라이트닝 토크로 발표한 분의 주제였다.
  위 링크를 들어가보면 16가지 패드가 있는데 이걸 실제로 구현하면 된다.
  좀 더 고도화를 하자면 신호의 강/약에 따른 소리변화까지..
