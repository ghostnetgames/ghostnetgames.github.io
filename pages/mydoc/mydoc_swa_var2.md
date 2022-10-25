---
title: 변수 이름의 기능
permalink: mydoc_swa_var2.html
keywords: 변수, 코드 작성법
sidebar: mydoc_sidebar
folder: mydoc
summary: 변수의 "수명"을 가능한 짧게 유지한다.
---

### 이름을 지을 때 가장 중요한 고려 사항
그 이름이 변수가 나타내는 것을 완전하고 정확하게 설명하는지를 가장 중요하게 고려해야한다.
미국 올림픽 대표팀 선수가 몇 명인지 나타내는 변수의 이름은 numberOfPeopleOnTheUsOlympicTeam이 좋다. 운동장의 좌석수는 numberOfSeatInTheStadium일 것이다. 올림픽에서 각 나라의 대표팀에서 획득한 최고점수를 표현하는 변수 이름은 maximumNumberOfPointsInModernOlympics일 것이다.  

이러한 이름에는 두 가지 특징이 있다. 
- 해석하기 쉽다.
- 너무 길 경우 실용적이지 않다.  

### 최적의 이름 길이
한 연구에서 변수 이름의 길이가 평균적으로 문자 `*10자에서 16자 사이*`일때 디버깅하기 위해서 들이는 노력을 최소화할 수 있다는 사실을 발견했다. 그렇다고 모든 변수의 이름을 가이드대로 작성해야한다는 의미는 아니다. 코드를 봤을 때 짧은 이름이 많다면 이름이 분명하게 작성되었는지 확인해봐야한다.

### 범위가 변수명에 미치는 효과
한 연구에 따르면 
- 긴 이름: 전역변수나 거의 사용하지 않는 변수
- 짧은 이름: 지역 변수나 반복문 변수{(i, z, k) → 이 변수가 반복문 밖에서도 활용된다면 의미를 갖는 이름을 할당해야한다.}  
가 좋다고 한다. 하지만 짧은 이름은 문제를 일으킬 수 있어 신중한 개발자들은 방어적으로 짧은 이름을 피한다.

### 변수 이름의 계산 값 한정자
많은 프로그램이 총계, 평균, 최댓값 등 계산된 값을 보관하는 변수를 갖는다.  
Total, Sum, Average, Max, Min, Record, String, Pointer와 같은 한정자로 변수 이름 끝에 한정자를 입력한다. (revenueTotal, expenseTotal...)  
관습적 예외 - Num  
Num 한정자는 변수의 이름 앞에 있으면 Num은 총계를 가리킨다. numCustomer는 전체 고객수이다. customerNum은 현재 고객의 번호이다. Num을 사용하면 혼란을 일으킨다. 전체 고객의 수를 가리키기 위해서 Count나 Total을 사용하고 특정 고객을 가리키는 데 Index를 사용하는 것이 좋은 방법이다.  

### 반복문 인덱스 이름
i, j, k와 같은 이름을 관습적으로 사용한다.
```c
for( i = 0 ; i < lastItem; i++) {
    data[i] = 0;
}
```
변수를 반복문 외부에서 사용해야한다면 반드시 i나 j, k보다는 좀 더 의미 있는 이름을 지어야한다.
```c
recordCount = 0;
while(moreScores()) {
    scroe[recordCount] = GetNextScore();
    recordCount++;
}
// recordCount를 사용하는 코드
```
배열 접근시에도 더 분명하게 한다. score[teamIndex][eventIndex]는 score[i][j]보다 많은 정보를 제공한다.

### 상태 변수 이름
상태 변수에 대해서 flag보다 더 나은 이름을 생각해 본다. flag라는 이름은 그것이 무엇을 하는지 아무런 단서도 제공하지 않기 때문이다.
```c
if(flag)...
if(statusFlag & 0x0f)
if(printFlag == 16)
if(computeFlag == 0)
```
statusFlag=0x80과 같은 명령문은 직접 코드를 작성했거나 statusFlag와 0x80이 무엇을 표현하는지 설명하는 문서를 보기 전까지 코드가 무엇을 하는지 전혀 알 수 없다.
```c
if(dataReady) …
if(characterType & PRINTABLE_CHAR) …
if(reportType == ReportType_annual) …
if(recalcNeeded == true) …

dataReady = true;
characterType = CONTROL_CHARACTER;
reportType = ReportType_Annual;
recalcNeeded = false;
```
확실히 더 이해하기 쉽다.   

### 임시 변수 이름
temp를 사용하지만 단순 swap이 아닌 의미있는 값을 담는 곳이라면 이름을 명시적으로 만들어야한다.

### 전형적인 Boolean 변수 이름을 기억한다.
- done : 무언가 수행되었다는 것을 가리키기 위해서 done을 사용한다. 완료이전에는 done을 false로 처리하고 완료후에는 true로 설정한다.
- error : error가 발생했을을 가리킨다. 오류가 발생하면 true, 오류가 발생하지 않았다면 false를 사용한다.
- found : 값이 발견되었다는 것을 의미한다.
- success 또는 ok : 연산이 성공적인지를 가리키기 위해 사용한다.

### 참이나 거짓의 의미를 함축하는 Boolean 변수의 이름을 사용한다.
done과 success와 같은 이름은 상태가 참 아니면 거짓으로 둘 중 하나이기 때문에 좋은 불린 변수 이름이다. 반면에 status와 sourceFile과 같은 변수는 참이나 거짓이 명백하지 않기 때문에 좋지 못한 불린이름이다. 좀 더 나은 결과를 위해서 statusOk나 sourceFileAvailable과 같은 이름으로 대체한다.

### 긍정적인 Boolean 변수 이름을 사용한다.
notFound, notDone과 같은 부정적인 이름은 이 변수의 값이 부정이 되었을 때 읽기가 어려워진다. 

### 열거형의 이름
Color_나 Planet_, Month_와 같은 접두사를 사용하여 해당 타입의 멤버가 같은 그룹에 속한다는 것을 보장할 수 있다. 그러나 어떤 프로그래밍 언어에서 열거형을 클래스처럼 다루고 있을 경우 (Color.Color_Red, Planet.Planet_Earth …) 반복하는 것이 아무런 의미가 없기 때문에 Color.Red와 같이 정의해야한다.

### 클래스와 객체 구별법
```c
// 방법1: 첫번째 문자를 대문자로 작성하여 타입과 변수를 구분함
Widget widget;
LongerWidget longerWidget;
// 방법2: 모든 문자를 대문자로 작성하여 타입과 변수를 구분함
WIDGET widget;
LONGERWIDGET longerWidget;
// 방법3: 타입에 대해서 "t_”라는 접두사를 넣어 타입과 변수를 구분함
t_Widget Widget;
t_LongerWidget LongerWidget;
// 방법4: 변수에 대해서 "a”라는 접두사를 넣어 타입과 변수를 구분함
Widget aWidget;
LongerWidget aLongerWidget;
// 방법5: 좀 더 구체적인 변수의 이름을 사용하여 타입과 변수를 구분함
Widget employeeWidget;
LogerWidget fullEmployeeWidget
```
방법1은 일반적인 규약이지만 첫번째 글자를 대문자로 구별하는 이름은 "심리학적인 거리"가 너무 가깝고 두 이름 사이의 외형상의 차이가 너무 작다.
그리고 특정 언어에서는 동일한 토큰으로 취급되기 때문에 문법 오류를 발생시킬것이다.  

방법2는 명확하게 구분이 가능하다. 하지만 여러 언어를 사용하는 경우 방법1과 같은 문제를 일으킨다.  
방법3은 모든 언어에서 효과가 있지만 호불호가 크다.  
방법4는 방법3 대신 사용되지만 클래스 … (이해가 안됨)  
방법5. 변수에 대해서 더 많이 생각해야한다. 대부분의 경우 변수에 더 구체적인 이름을 지으려고 노력하면 더 읽기 쉬운 코드가 작성된다.  
모든 방법에는 trade-off가 따르지만 저자는 방법 5를 선호한다.  


{% include links.html %}
