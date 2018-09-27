---
title: "Vimrc Color 적용하기"
date: 2018-09-27 14:20:28 +0900
categories: vim vimrc development
---

## .vimrc 파일

```bash
# .vimrc
set autoindent # 자동 들여쓰기
set smartindent # 또 자동 들여쓰기
set wrap # 줄바꿈 표시
set nocompatible # vim 디폴트 기능 사용
set nobackup # 백업 파일을 만들지 않음
set ruler # 우측 하단에 현재 커서 위치 표시
set shiftwidth=4 # 자동 들여쓰기 간격
set nu # set number 줄번호 표시
set tabstop=4 # 탭 간격
set ic # set ignorecase 검색 시 대소문자 구분 안함
set tenc=utf-8 # 터미널 인코딩
set backspace=eol,start,indent # 줄의 끝, 시작, 들여쓰기에서 백스페이스 누르는 경우 이전 줄로 이동
set history=500 # 편집 기록 갯수, .viminfo
syntax on # 자동 문법 강조
filetype indent on
colorscheme sublimemonokai
```

참고
[vim 설정을 위한 .vimrc](https://blog.outsider.ne.kr/518)
[.vimrc 설정 파일-1](https://opentutorials.org/course/730/4578)

<hr>

## color 테마 적용
