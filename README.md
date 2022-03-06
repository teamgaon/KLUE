# [한국어 문장 관계 분류 경진대회](https://dacon.io/competitions/official/235875/overview/description) 🏆 [Private, 1st]


![image](https://github.com/teamgaon/KLUE/blob/main/pic/16.png)


## Data
* premise와 hypothesis의 관계를 판별하는 모델을 개발
  + Train
![image](https://github.com/teamgaon/KLUE/blob/main/pic/1.png)
  + Test
![image](https://github.com/teamgaon/KLUE/blob/main/pic/2.png)

* 외부데이터
  + [KAKAO](https://github.com/teamgaon/KorNLUDatasets)<br>
  ![image](https://github.com/teamgaon/KLUE/blob/main/pic/3.png)
  + [KLUE](https://klue-benchmark.com/tasks/68/data/description)
  ![image](https://github.com/teamgaon/KLUE/blob/main/pic/4.png)

## EDA
* KLUE Data Label
  + 5개의 label을 토대로 gold_label을 판별
  ![image](https://github.com/teamgaon/KLUE/blob/main/pic/5.png)
* premise와 hypothesis, label의 관계
  + 대체로 하나의 premise에 hypothesis 세 문장, 세 개의 label이 매칭
  ![image](https://github.com/teamgaon/KLUE/blob/main/pic/6.png)
* label 분포
  + 세 개의 label이 고르게 분포<br>
  ![image](https://github.com/teamgaon/KLUE/blob/main/pic/7.png)
  
## 모델링
5 fold로 모델을 fine-tuning 후 Voting ensemble
* Model
  + [Roberta-large](https://huggingface.co/klue/roberta-large)
* Optimizer
  + [AdamW](https://pytorch.org/docs/stable/generated/torch.optim.AdamW.html)
* Learning rate scheduler
  + [Cosine annealing with warmup](https://huggingface.co/docs/transformers/main_classes/optimizer_schedules#transformers.get_cosine_schedule_with_warmup)

## 추론
* Hard voting
  + 5개의 모델로 label을 예측
![image](https://github.com/teamgaon/KLUE/blob/main/pic/8.png)
  + 대부분의 모델이 neutral에 치중<br>
![image](https://github.com/teamgaon/KLUE/blob/main/pic/9.png)
  + 5개의 label 중 같은 label이 3개 이상이면 해당 label 획득, 아니면 neutral
    ```
    def voting(df):
      cols = ['neutral', 'contradiction', 'entailment']
      for col in cols:
        if df[col] > 2:
          return col
      return 'neutral'
    ```
  + 가장 빈번한 label을 부여했을 때 label이 neutral에 치중 <br>
![image](https://github.com/teamgaon/KLUE/blob/main/pic/10.png)
  + label 별로 threshold 다르게 적용
    ```
      neutral: 3 / contradiction: 2 -> contradiction
      neutral: 3 / entailment: 2 -> entailment
      neutral: 4 / contradiction: 1 -> contradiction
      neutral: 4 / entailment: 1 -> entailment
    ```
   + 결과 <br>
![image](https://github.com/teamgaon/KLUE/blob/main/pic/11.png)
* Soft voting
  + 추가 
