---
title: "Trouble shoot pandas SettingWithCopyWarning"
date: 2020-01-31
---

## How to solve SettingWithCopyWarning

회사에서 사용 중인 파이썬 스크립트를 분석할 일이 생겨 분석하던 중 다음과 같은 경고문을 만났습니다.

![warning](/images/2020-01-31-trouble-shoot-pandas-setting-with-copy-warning/0.png)

도대체 왜 이런 문구가 떴는지를 알기 위해 이전에 실행되었던 셀을 함께 살펴보았습니다.

![notebook-cell](/images/2020-01-31-trouble-shoot-pandas-setting-with-copy-warning/1.png)

`local_wmp_df` 라는 DataFrame에 특정 조건을 만족하는 로우들만 추출하기 위해 위와 같은 구문을 사용했었습니다. 

위 구문의 문제점은 원본 DataFrame을 그대로 사용하고 싶은 건지, 아니면 복사본을 생성하여 다루고 싶은지가 불명확 했고 그로 인해 경고 메시지가 나온 것이었어요. :chicken:

DataFrame을 다룰 때는 copy를 다루는 것인지 아니면 원본을 다루는 것인지를 항상 명시적으로 기입해줘야 한다는 걸 알았습니다. 이는 Deep Copy와 Shallow Copy 중 어떤 것을 선택할 것이냐의 문제라는 것두요. :white_check_mark:

제가 필요했던 것은 원본 DataFrame에 대한 변경이 아닌 Deep Copy를 톡한 복사 및 그에따른 작업이었기 때문에 명시적으로 `.copy()` 를 사용하여 복사를 해주었습니다.

만약 Shallow Copy로 원본 DataFrame에 대해 뭔가 변경을 하고 싶었다면, `inplace=True` 옵션을 사용해줘서 처리할 수도 있습니다.

파이썬도 고수준 언어로 굉장히 사용하기 편리하긴 하지만, 결국 저수준 언어에서 요구하는 Shallow Copy, Deep Copy와 같이 기본 CS 지식이 필요한 언어이고 역시 기본기가 중요하구나 라는 것도 다시 상기하는 계기가 되었습니다. :muscle: :eyes: :fist_right: