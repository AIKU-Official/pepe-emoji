## 소개

> 내 상황에 딱 맞는 이모지가 없다고요? 걱정 마세요! 직접 만들어 드립니다!
> 

요즘 카카오톡에서 이모지 없이 대화하기 어려운 분들이 많습니다. 말끝마다 이모지를 붙이지 않으면 왠지 허전한 느낌이 드는 경험, 공감하시죠? 하지만 현재 애플 이모지를 기준으로 총 3,782개의 이모지가 제공되더라도, 다양한 상황과 감정을 모두 표현하기엔 한계가 있습니다.

이를 해결하기 위해 애플은 Apple Intelligence 기능 중 하나로 젠모지(Genmoji)를 도입했습니다. 젠모지는 스타일, 상황, 감정 등을 직접 지정해 나만의 이모지를 생성할 수 있는 기능입니다. 실제로 사용해보니 기존에 없던 이모지를 쉽게 만들 수 있어 흥미로웠습니다. 하지만 이 기능은 아이폰 15 이상에서만 사용할 수 있으며, 강력한 필터링이 적용되어 있어 약간이라도 부적절할 가능성이 있는 내용은 절대 생성되지 않는다는 단점이 있습니다. 게다가 한국에서는 아직 지원되지 않는 상황입니다.

이에 저희는 누구나 자유롭게 이모지를 생성할 수 있는 **나만의 이모지 만들기** 프로젝트를 기획했습니다. 이 프로젝트를 통해 기기나 플랫폼에 제한받지 않고, 원하는 감정과 상황을 보다 자유롭게 표현할 수 있는 모델을 만들었습니다.

## 목표

애플 젠모지가 여러가지 스타일을 제공하는 것과 달리 저희는 상황과 감정에 따른 커스타마이징에 집중하기 위해 페페 스타일의 한 가지 이모지만 선택했습니다.

- 기존 이미지가 표현할 수 없는 상황, 감정을 10개 이상 표현

## 선행 사례

SDXL-emoji

- 텍스트 프롬프트로 애플 스타일의 이모지를 생성할 수 있는 모델
- Dreambooth와 LoRA를 이용해 SDXL을 파인튜닝
    
![image](https://github.com/user-attachments/assets/a0fd7c39-d234-47be-b456-794c4542c716)

    
- `A TOK emoji of a man`

## 데이터셋

| **데이터셋** | **설명** |
| --- | --- |
| [페페 데이터셋](https://www.kaggle.com/datasets/tornikeonoprishvili/pepe-memes-dataaset) | 여러 가지 페페 짤들이 담긴 데이터셋 |
- 데이터셋 특성 상 이모지가 아닌 짤로 볼 수 있는 데이터가 일부 포함(배경)
    - 이모지 느낌의 이미지만 모아둔 재미있는 데이터셋이 없음
    - 해당 데이터셋에 단색 배경의 이미지가 더 많음
    - 따라서 그냥 이용하기로 결정

## 모델링

- 베이스 생성 모델로 Stable diffusion 1.4 사용
    - titan XP의 12GB VRAM으로 충분한 모델
- LoRA 및 Dreambooth 기법을 사용
- 다양한 방식으로 데이터 어노테이션 후 파인튜닝 적용
    - 데이터셋의 제목 그대로 사용 (20~30개, 2300개)
    - gpt를 이용해서 이미지에 대한 description을 얻고 label로 활용(150개)

## 훈련 및 평가

### 훈련

- Dreambooth, LoRA를 이용해 Stable diffusion 1.4 학습
- huggingface peft 라이브러리 이용
- batch size: 1, lr: 5e-6
- emoji 데이터를 구하기 애매해서 prior preservation은 하지 않

### 데이터

- 원본 데이터 - 페페 이미지 2300개 + 간단한 description 제목
    
  ![image](https://github.com/user-attachments/assets/93cb6ea4-d020-4e8d-8365-bab4efc09fcf)

    
- 1차 시도: 이미지 30개 + description 수작업
    - 이모지 스타일 (배경 x) 이미지 선별하여 진행
- 2차 시도: 이미지 2300개 + 제목의 description 추출
- 3차 시도: 이미지 150개 + gpt 이용 description 생성

### 평가

- style을 바꾸는 dreambooth 학습 특성 상 Loss 가 큰 의미 없음
- epoch(training step)을 바꾸며 정성적 평가

## 결과
![image](https://github.com/user-attachments/assets/ccc0b272-3365-4493-8699-f4086e1ec26f)


- 이미지 2300개, 50epoch (100000 training steps)의 결과가 가장 잘 나옴
- 간단한 프롬프트에 대해 어느 정도 맥락에 맞는 이미지 생성 가능
- 대부분 단색 배경이 출력되지만 그렇지 않은 경우도 있음

## 한계 분석

- 정량적 평가 부재
- 복잡한 프롬프트를 잘 수행하지 못함
- 다리가 세 개가 되는 등 비정상적인 결과가 나오기도 함
- 페페가 아닌 다른 객체는 표현 불가

## 후속 프로젝트

- 다양한 스타일로 이모지 생성기 제작
- 정량적 평가 지표 도입(CLIP score 등)
- prior preservation 활용해 결과물 개선

## 데모

```bash
pip install -r requirements.txt
cd ./peft/examples/lora_dreambooth_test2
python3 /home/aikusrv02/aiku/Emoji/peft/examples/lora_dreambooth_test2/demo.py
```
