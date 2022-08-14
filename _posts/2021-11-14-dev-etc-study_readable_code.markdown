---
layout: post
title:  "[Book] The Art of Readable Code 1"
subtitle:   "Subtitle - The Art of Readable Code"
categories: dev
tags: etc
comments: true
published: true
---
## Describe
> The Art of Readable Code 읽고 정리1<br>

## 목차 
- [Describe](#describe)
- [목차](#목차)
- [<a name="jump1">이름을 짓는 방법</a>](#이름을-짓는-방법)
- [<a name="jump2">오해할 수 있는 이름들</a>](#오해할-수-있는-이름들)
- [<a name="jump3">보기좋게 코드 쓰기</a>](#보기좋게-코드-쓰기)

<br><br>

## <a name="jump1">이름을 짓는 방법</a>
1. 명확한 이름을 짓자.<br>알고 있는 단순한 영어표현을 쓸 게 아니라, 구체적인 의미를 갖고 있는 단어를 사용하자. 

    |단어|대체어|
    |---|---|
    |send|deliver, dispatch, announce, route|
    |find|search, extract, locate, recover|
    |start|launch, create, begin, open|
    |make|create, setup, build, generate, add, new|
2. 범용적인 이름을 피하자.<br>retval, tmp, i/j/k와 같이 습관적으로 사용해버리는 이름에 조금 더 고민하자. 되도록이면 역할을 잘 표현하는 이름을 사용하고, i/j/k의 경우, 같이 사용되는 리스트의 이름에 맞춰두면 버그를 찾기에도 좋다.
    ```python
    # example of retval and i/j/k
    def return_norm2(num, index_list):
      retval = 0
      for i in index_list:
        retval += num[i] * num[i]
      return Math.sqrt(retval)

    def return_norm2(num, index_list):
      sum_squares = 0
      for ni in index_list:
        sum_squares += num[ni] * num[ni]
      return Math.sqrt(sum_squares)

    # example of i/j/k
    clubs[i].members[j] = users[k]

    clubs[ci].members[mi] = users[ui]
    ```
    tmp와 같은 경우
    ```python
    tmp = right
    right = left
    left = tmp
    ```
    정도로 잠깐만 사용하는 경우에는 괜찮지만, 그 이외의 역할을 한다면, 다른 이름을 고려하는 것이 좋다. 예를 들면, tmp_filename, tmp_filepath 와 같이 변수의 특정 역할을 알기 쉽게 표시
3. 수치를 표시하자.<br>단위를 사용할 수 있는 변수는 다양하다. 가능한 구체적인 정보를 전달하자.
   - delay_secs
   - size_mb
   - degree_mm
   - length_char
   - length_text
4. 변수의 주요 속성을 표시하자. <br>버그가 될 수 있는 상황을 변수의 이름으로 막을 수 있다. 
   - plaintext_password
   - unescaped_comment
   - html_utf8
5. 이름의 길이에 대해서. <br>생성 후 바로 사용되는 변수이거나, 그 의미가 몇 줄 안에 파악되는 변수라면, 크게 신경쓰지 않아도 된다. 이외의 상황에서는, **누가봐도 이해가 되는** 수준 안에서 이름에 충분한 의미를 부여한다. 예외로
   - ConvertToString : ToString
   - DoServerLoop : ServerLoop
  
    와 같이, 일반적으로 사용/이해할 수 있는 부분은 짧게 쓰자.
   
## <a name="jump2">오해할 수 있는 이름들</a>
1. 애매한 의미를 가질 수 있는 이름들
   |의도|오해|제안|
   |---|---|---|
   |filter(condition)|condition을 만족하는 dataset이 리턴되는가? 그 반대인가?|search(),exclude()
   |read_password(boolean)|읽을 필요가 있는 것인지, 이미 읽은 것인지?|is_authenticated|
2. 오해를 줄일 수 있는 표현들
  - max, min : 어떤 파라미터의 한계를 표현할 때
  - first, last : first와 last를 포함한 처리
    ```python
    print_name(first='Han', last='Kim')
    ```
  - begin, end : 이벤트 기간과 같은 범위
    ```python
    print_events(begin='OCT 16 12:00am', end='OCT 17 12:00am')
    ```
3. 다른 유저들의 기대치에 맞춘 이름작성
  - get()<br>get함수는 일반적으로 클래스 맴버가 갖고 있는 수치를 단순히 반환하는 역할을 한다. getMean()이라는 함수에 복잡한 load/calculate 등이 포함되면 안되겠다.
  - size() : get()과 비슷하게, O(1)시간이 걸리는 것이 일반적이다.

## <a name="jump3">보기좋게 코드 쓰기</a>
1. 비슷한 역할을 하는 코드는 비슷하게 쓰기
2. 반복을 피하기 위한 함수를 작성하기
   ```python
    def expand_full_name(database, partial_name):
      # Search full_name from the database
      # Return error_code, if it occurs an error
      return full_name, error_code 

    def check_full_name(partial_name, expected_full_name, expected_error):
      full_name, error = ExpandFullName(dataset, partial_name)
      assert(error == expected_error)
      assert(full_name == expected_full_name)
    
    check_full_name('Han',        'Han Beomseok',   '')
    check_full_name(' Nhand',     'Nhandsome',      '')
    check_full_name('Not exist',  '',               'no match found')
    check_full_name('Kim',        '',               'more than one result')
   ```
   열을 맞추는 것으로 오탈자를 확인하는 것도 쉬워진다.
3. 논리적인 흐름을 코멘트로 작성하기
   ```python
    def suggest_new_friends(user, emali_password):
      # Read all friends of user
      friends = user.friends()
      ...

      # import all emails from user email account
      contacts = import_contacts(user.email, email_password)
      ...

      # Search non friend emails among contact emails
      non_friend_emails = contacts - friends
      ...

      # Make lists of information to display
      ...

      return display_suggest_lists
   ```
4. 정리하는 것에 답은 없지만, 본인의 코드 내에서는 일관성을 지킨다. 

