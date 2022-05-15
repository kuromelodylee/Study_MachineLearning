1. **케라스란 무엇인가?**

   - 딥러닝 알고리즘을 구현하게 해주는 도구
   - 텐서플로우, 파이토치, 케라스 순으로 많이 쓰임

2. **케라스 장단점**

   - 장점: 간단하다. (텐서플로우와 같이 저수준 라이브러리를 API화 한 것임)
   - 단점: 자유도가 떨어진다. (API화 되어있기때문에

3. **케라스 사용법**

   데이터 생성 -> 모델 프레임 생성 -> 모델 층 정의 -> 모델 학습 준비 -> 학습 

   1. *데이터 생성*
      - 자신의 데이터를 준비한다.
      - kearas 기본 API에 데이터 세트가 있다. 간단히 from keras.datasets로 import할때 불러오면 된다.
      - https://keras.io/api/datasets
   2. *모델 프레임 생성*
      - Sequantial mode:  model = Sequantial() 선언하고 model.add를 통해서 컴파일 전까지 레이어를 적층해간다. 
        - https://keras.io/api/models/sequential/
      - Model mode: 모델을 input부터 output까지 각각레이어를 선언하고 Model(input, output으로 모델프레임을 생선한다.
        - https://keras.io/api/models/model/#model-class
   3. *모델 층 정의 (프래임 안에서)*
      1. 모델 프레임 내의 각각의 모델 층(레이어)에 대해서 정의 한다.
         - [Dense layer](https://keras.io/api/layers/core_layers/dense)
         - [Activation layer](https://keras.io/api/layers/core_layers/activation)
         - [Conv2D layer](https://keras.io/api/layers/convolution_layers/convolution2d)
         - [LSTM layer](https://keras.io/api/layers/recurrent_layers/lstm)
         - whole layers api manual: https://keras.io/api/layers/
   4. *모델 학습 준비 model.complie*
      1. 준비한 모델프레임과 모델 레이어들이 어떤방법으로 학습할 것인가를 결정한다.
         - https://keras.io/api/optimizers/
      2. 컴파일의 인자는 optimizer, loss(손실함수), metrics(측정항목)를 기본으로 많이 사용한다.
         - https://keras.io/ko/optimizers/ & https://keras.io/api/optimizers/
         - https://keras.io/ko/losses/ & https://keras.io/api/losses/
         - https://keras.io/ko/metrics/ & https://keras.io/api/metrics/
   5. *모델 학습 model.fit* 
      1. 컴파일까지 준비한 모델에 학습데이터들을 넣어 주어서 학습을 시작한다.
      2. fit의 인자는 (학습 데이터, 학습 라벨, 배치사이즈, 이포크사이즈)
         - 정의한 모델의 파라미터 값들을 여러 차례 갱신하면서 최적의 파라미터를 찾아야 하므로 학습과정을 여러번 해야한다.  이를 위해 배치사이즈 및 이포크라는 개념이 필요하다.
         - 배치사이즈(batch size): 한번 모델로 학습하는 데이터 셋의 크기
         - 이포크(ephoch): 전체 데이터셋을 몇 번 학습할 것인가

   6. *예측 model.predict*
      1. 학습이된 모델에 test 데이터를 넣어서 예측 한다.
