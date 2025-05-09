---
layout: post
title: LLaMA3 기반 디지털포렌식 아티팩트 분석기 개발 (1)
author: leewoojin
date: 2025-05-07 00:00:00 +0900
categories: [Forensics, LLaMA3]
tags: [llm, llama3, forensic, automation, artifact]
---

## 1. 파일 다운로드

개인 컴퓨터 사양에 맞는 파일을 아래 사이트에서 다운로드합니다.

🔗 [https://ollama.com/download](https://ollama.com/download)

![ollama 다운로드 페이지](https://github.com/user-attachments/assets/238737a4-72be-447c-a99b-1e875699e1b2)


## 2. 다운로드한 파일 실행

설치 파일을 실행하면 아래와 같은 화면이 순서대로 나타납니다.

<div style="display: flex; gap: 10px; justify-content: space-between;">
  <figure style="text-align: center; width: 30%;">
    <img src="https://github.com/user-attachments/assets/15b22885-7261-497c-a3c0-ab49670b130d" alt="설치 시작" style="width: 100%;">
  </figure>

  <figure style="text-align: center; width: 30%;">
    <img src="https://github.com/user-attachments/assets/a0e5f8c3-4338-46a9-b2bc-d716911b8e82" alt="설치 중" style="width: 100%;">
  </figure>

  <figure style="text-align: center; width: 30%;">
    <img src="https://github.com/user-attachments/assets/bcc617ff-11b6-44b6-8167-4038d6a4f949" alt="설치 완료" style="width: 100%;">
  </figure>
</div>

## 3. LLaMA3 모델 실행

설치가 완료되면, 터미널에서 아래 명령어를 복사하여 붙여넣고 실행합니다:
```bash 
ollama run llama3
```
> `llama3.2`와 같은 최신 버전 사용을 권장하지만, 저는 추후 Google Colab과 연동할 계획이 있어 기본 `llama3` 모델을 설치해 사용했습니다.

