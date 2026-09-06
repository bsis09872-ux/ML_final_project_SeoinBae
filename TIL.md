## 과제 진행 로그
1. uv init
2. uv add pandas
3. source .venv\bin\activate (mac os 기준)
    - .ps1 => windows powershell
    - .bat => windows cmd

4. uv add numpy matplotlib seaborn scikit-learn ipykernel jupyter
    - jupyter는 자체적으로 설치해도 됨. 나는 그냥 한번에 설치함.
    - .toml에 알아서 업데이트됨. 나중에 uv sync로 환경 통일 가능.

5. EDA & 데이터 전처리 파이프라인 구현 (단계 1) 시작 🐌
    - 필요한 툴 불러오기
    - 폰트 깨짐 방지 (추가)
    - 파일 불러오기 (read_csv())
    - 기본적인 확인 시작
        1. head()
        2. info() -> NON-NULL 상에는 누락된 값 없음.
        3. isnull().sum() -> NULL값 없음.
        4. shape

    - 데이터셋 기본 정보 정리 (마크다운) : 각 항목이 어떤 의미인지 확인 및 명시
    - 결측치와 이상치 확인 및 처리
        1. describe() -> 0일 수 없는 항목에 0이 있음 (결측치 의심)/ 최댓값이 너무 큼 (이상치 의심)
            - 임의로 하나의 컬럼만 히스토그램을 그려봄.
            - 그 외 다른 컬럼에 대해서도 mean 과 median의 크기 비교를 통해 간단히 편향된 분포인지, 어느쪽으로 편향되었는지 알 수 있다.
        2. (df == 0).sum() : 값이 0으로 처리된 값의 개수 확인
        3. 0일 수 없는 항목에 대해서 NaN 결측치 처리함.
        4. NaN 처리한 결측치에 대한 처리 판단
            
    | 방법 | 언제 사용? | 대표 상황 | 현재 데이터 |
    |---|---|---|---|
    | **평균(Mean)** | 분포가 비교적 대칭이고 이상치가 적을 때 | 키, 시험점수 등 | △ |
    | **중앙값(Median)** | 치우친 분포·이상치가 있을 때 | 소득, 의료 수치 등 | **⭕ 유력** |
    | **ffill** | 순서/시간이 있고 직전 값이 의미 있을 때 | 시계열 | ❌ |
    | **bfill** | 순서/시간이 있고 다음 값이 의미 있을 때 | 시계열 | ❌ |
    | **선형보간(Interpolation)** | 앞뒤 관측치 사이의 연속적 변화가 의미 있을 때 | 시간별 온도 등 | ❌ |
    
        - 현재 데이터는 이상치가 존재하고 plot을 그려보면 right-skewed long-tailed 분포인 것을 확인할 수 있다. (정규분포 아님. 이상치 있음.) -> 평균을 사용하기에 어려워보임
        - 이러한 경우 이상치의 영향을 비교적 적게 받는 중앙값으로 **통계적 대푯값 대치**가 가능해 보임
        - 그외 ffill/bfill은 시계열 데이터가 아니라 해당되지 않으며, 선형 보간법 또한 연속적 변화의 의미가 있는 데이터에 해당되지 않으므로 고려 대상이 아님.

        5. 결측치(NaN) -> 중앙값으로 대치: df[col].median()
            - 컬럼이 여러개라 for문 사용
            - <주의> 대치 처리 후 반드시 사용하는 데이터프레임에 반영하기 (df['col']=...)
        
        6. 중복 데이터 확인 및 처리 : duplicated(keep=False)
            - 중복값 없어서 별도 처리 없음
        7. [심화] 상관관계 확인 → Outcome과 관련성이 높은 변수 선정 
        8. 주요 변수 분포 시각화

6. 머신러닝 모델 학습, 과적합 점검 및 하이퍼파라미터 튜닝 (단계 2) 시작 🐢


7. Hugging Face 사전 학습된 모델을 활용한 뉴스 기사 분류 및 추론 (단계 3) 시작 🐰


---
## 새롭게 공부한 내용:
### 범주형 변수 인코딩 (Categorical Encoding)

- **범주형 변수(Categorical Variable)**: 숫자의 크기보다 특정 **범주(Category)** 를 나타내는 변수
  - 예: 성별, 혈액형, 지역, 당뇨병 여부
- 대부분의 머신러닝 모델은 문자열을 그대로 계산하기 어려우므로, 범주형 데이터를 **숫자 형태로 변환하는 과정**이 필요하다.
- 이를 **범주형 변수 인코딩(Categorical Encoding)** 이라고 한다.

#### 1. Label Encoding
각 범주를 하나의 숫자로 변환한다.
```
Female → 0
Male   → 1
```
```
from sklearn.preprocessing import LabelEncoder

encoder = LabelEncoder()
df['Gender'] = encoder.fit_transform(df['Gender'])
```
#### 2. One-Hot Encoding
각 범주를 별도의 컬럼으로 만들어 0과 1로 표현한다.
```
Color     Red   Blue   Green
Red        1      0      0
Blue       0      1      0
Green      0      0      1
```
```
df = pd.get_dummies(df, columns=['Color'])
```