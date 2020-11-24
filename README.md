# SW_Microprocessors_Capstone_Design_Lab
 
## 1. 설계목적
- PIC16F876A를 이용하여 디지털 시계를 제작하기  
- 어셈블리어를 이용한 알고리즘 구현하기

## 2. 시스템 동작 내용(HW/SW)
### 2-1. HW 구성 및 동작

![1](https://user-images.githubusercontent.com/58457978/100061682-c5d22d80-2e71-11eb-9c17-b98691cb2e93.png)   
그림 1 전체 회로 구성 (전면, 후면)

|SW A|SW B|SW C|
|---|---|---|
|RB 3|RB 4|RB 5|  

표 1 스위치와 PIC 칩의 연결도

|RC 7|RC 6|RC 5|RC 4|RC 3|RC 2|RC 1|RC 0|
|---|---|---|---|---|---|---|---|
|dp|g|f|e|d|c|b|a|  

표 2 세그먼트와 PIC 칩의 연결도

|DG 1|DG 2|DG 3|DG 4|
|---|---|---|---|
|RA 3|RA 2|RA 1|RA 0|  

표 3 각 digit와 PIC 칩의 연결도

### 2-2. SW 구성 및 동작
- 전체적인 프로그램은 기본적으로 시간을 생성하는 메인 루틴을 수행하고, 그 사이에 TMR0 overflow 인터럽트가 발생하면 디스플레이(DISP) 할 단자를 결정하며 버튼입력(BTN_CHK)을 확인하여 작동하고자 하는 Flag를 변경한다. 인터럽트가 발생하지 않았을 때에는 지정된 Flag를 확인하여 디스플레이 값을 유지하거나 변경하도록 구성되어있다.

1) 스위치 동작
- 인터럽트 수행 중 버튼입력을 확인하고 일정 시간동안 지속된 버튼입력을 수행하도록 하였다. 

2) 디스플레이
- 인터럽트가 수행되면 FLAG의 하위 2비트를 이용해 4개의 분기로 나누어 출력될 DIGIT을 선택하도록 설계하였다.

3) 시간 구현
- TMR0와 내부 클럭과 프리 스케일러를 이용하여 타이머를 작동시키고, TMR0의 오버플로우를 이용한 인터럽트가 4회 발생시 시간 측정을 위한 카운트를 1 증가시키고, 이 카운트가 122회 반복되면 1초가 증가하게 하였다. 1초 카운트가 10이 되면 10의 자리 초가 증가하고 그 초가 7이 되면 1분의 자리가 1 증가한다. 이를 반복해 24시까지 표현되도록 하였다.

4) 각각의 모드와 작동
- 모드는 가장 기본이 되는 시간표현(시간:분), 초만 표시하는 모드, 시각설정모드가 있다.
시간:분(00:00)을 표시하던 상태에서 초(XX:00)만 표시하는 상태로 다시 현재시각 설정모드로 이동할 수 있다. 각각의 모드들은 SW A의 동작으로 MENU_FLAG를 변화시켜 접근 가능하며 시간설정 모드에서는 SW B를 이용해 DIGIT을 변경하고, SW C를 이용해 각 DIGIT의 인수들을 증가시킬 수 있다.

## 3. 기능 및 동작과정
인터럽트 및 MENU_FLAG를 이용한 분기

```assembly
   ORG   0004H

   MOVWF   W_TEMP
   
   SWAPF   STATUS,W
   MOVWF   STATUS_TEMP
   MOVLW   B'00000000'
   CALL   DISP
   CALL   BTN_CHK
   SWAPF   STATUS_TEMP,W
   MOVWF   STATUS
   SWAPF   W_TEMP,F
   SWAPF   W_TEMP,W
   BCF   INTCON,2
   
   RETFIE
   
   CHK_FLAG
   BTFSC   MENU_FLAG,0
   GOTO   CLK_MODE      ;CLK MODE OR SETTING
   
   CLK_MODE
   
   BTFSC   MENU_FLAG,7
   GOTO   CLK_MODE_SECOND
   BTFSC   MENU_FLAG,4
   GOTO   SETTING_MODE_1
```
