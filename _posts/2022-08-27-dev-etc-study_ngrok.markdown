---
layout: post
title:  "[Memo]Ngrok이용해서 Colab환경 VS-code에서 사용하기"
subtitle:   "Subtitle - Colab with Ngrok"
categories: dev
tags: etc
comments: true
published: true
---
## Describe
> Memo, Colab, Ngrok<br>


## 사용하기 이전에...
- [ngrok](https://ngrok.com/)가입하기
- NGROK_TOKEN 확보하기

## Google Drive / Colab 환경을 VS CODE에서 사용하기

1. Colab - Google Drive 마운트하기
2. Colab에서 아래 코드 실행

```python
NGROK_TOKEN = 'ngrok 로그인 후 홈페이지에서 가져오기'
PASSWORD = '1234' # 임의설정

!pip install colab_ssh --upgrade
!chmod -R 777 ngrok

from colab_ssh import launch_ssh
launch_ssh(NGROK_TOKEN, PASSWORD)
```

- 실행결과 아래 ssh config 얻음

```python
[Optional] You can also connect with VSCode SSH Remote extension using this configuration:

  Host google_colab_ssh
    HostName xxx.xxx.ngrok.io
    User root
    Port xxxx
```

3. VS CODE에서 위의 config 로 ssh 접속가능
  - Open ssh config file
  - Connect to host
  - 접속합니까? Continue
  - 암호는? 위 Colab에서 입력한 임의 암호
  - 새로운 폴더 열면 '/content/drive/MyDrive/...'와 같이 Google Drive파일 사용가능
  - Colab GPU도 사용가능